**Category**: Reverse Engineering  
**Flag**: `VSL{9357be37ff33a3dabaf5998ad6903087}`

---

## 💡 Solution

This challenge involves reverse engineering to extract the login credentials. After logging in, the goal is to interact with the chatbot to retrieve the flag.

### Step 1: Extracting Login Credentials
Analyzing the `main_loginHandler` function using IDA reveals the following logic:

```assembly
if ((char *)qword_BBA0A8 == "username")
    v7 = runtime_memequal();
else
    v7 = 0;
if (!v7
    || ((net_http___Request__FormValue(), (char *)qword_BBA0B8 == "password") ? (v8 = runtime_memequal()) : (v8 = 0), !v8))
{
    runtime_makemap_small();
    runtime_mapassign_faststr();
    *v9 = &unk_7DF940;
    if (runtime_writeBarrier)
    {
        v9 = (_QWORD *)runtime_gcWriteBarrier1();
        *v11 = v10;
    }
    v9[1] = &off_91B750;
    if (v23)
    {
```

By inspecting the memory segments referenced in this function, the following credentials are found:

```assembly
.data:0000000000BBA090
.data:0000000000BBA0A0                               public main_username
.data:0000000000BBA0A0 E0 92 86 00 00 00 00 00       main_username dq offset aAdmin          ; "admin"
.data:0000000000BBA0B0                               public main_password
.data:0000000000BBA0B0 16 9E 86 00 00 00 00 00       main_password dq offset a123456         ; "123456"
```

The extracted credentials are:
- **Username**: `admin`
- **Password**: `123456`

These credentials allow successful login.

### Step 2: Interacting with the Chatbot

The `main_chatbotResponse` function contains the logic for processing messages. Key portions of the code include:

```assembly
*(_OWORD *)v63 = v4;
sub_476E2B();
LODWORD(v63[0]) = 1732584193;
*(_QWORD *)((char *)v63 + 4) = 0x98BADCFEEFCDAB89LL;
*(_QWORD *)((char *)&v63[1] + 4) = 0xC3D2E1F010325476LL;
*(_OWORD *)&v63[11] = v4;
runtime_stringtoslicebyte();
crypto_sha1___digest__Write();
v5 = crypto_sha1___digest__Sum();
```

The response is validated using an SHA-1 hash comparison:

```assembly
if (v15 == a1 && (unsigned __int8)runtime_memequal())
{
    // Decrypt using XOR
    for (i = 0LL; i < v21; ++i)
    {
        if (!v18)
            runtime_panicdivide();
        v52 = v18;
        v53 = i % v18;
        if (v53 >= v52)
            runtime_panicIndex();
        *(_BYTE *)(v17 + i) = *(_BYTE *)(v19 + v53) ^ v20[i];
        v18 = v52;
    }
}
```

### Step 3: Identifying the Secret Key
The `main_sendMessageHandler` function passes the secret key as a parameter:

```assembly
v15 = main_chatbotResponse(qword_BBA0C8);
```

From earlier analysis, the secret key is stored as:

```assembly
.data:0000000000BBA0C0                               public main_secret
.data:0000000000BBA0C0 5C FB 87 00 00 00 00 00       main_secret dq offset a5eab7a25fdc1cb   ; "5eab7a25fdc1cbc959eb8378386b557adbb23265"
```

The secret key is `5eab7a25fdc1cbc959eb8378386b557adbb23265`. Using an online tool like `crackstation.net`, we can decrypt the SHA-1 hash to reveal the plaintext: `querty`.

### Step 4: Retrieving the Flag
To retrieve the flag, send a correctly hashed message to the bot. The bot will validate the hash and respond with the flag.

