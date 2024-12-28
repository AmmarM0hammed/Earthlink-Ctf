# Earthlink CTF
## Web Category
Challange Name : [ Bi Database Analyst ]
Poinst : 80
Category : WEB
#### Challenge Description
The challenge provided a login page that authenticated users via cookies. Upon analyzing the application, it was found that manipulating the cookie value granted access to a page with SQL query execution capabilities. The goal was to retrieve the contents of a `.txt` file stored on the server.

**Exploiting the Vulnerability:**
Used the following SQL command to retrieve the contents of the desired text file:

   `RUNSCRIPT FROM 'file:/tmp/flag.txt';`

The server executed the command and returned the content of the file, which included the flag. 

> `CTF{HERE_U_GO_H2_JDBC_ATTACK}`

Challange Name : [  The Power Of cloud DeveOps ]
Poinst : 100
Category : Misc

#### Challenge Description
The challenge provided the following Azure Storage Blob URL:

`https://earthlinkctf.blob.core.windows.net/$web/`

> The URL pointed to the `$web` container in the Azure Storage Blob
> service, commonly used to host static websites.

If a file had been removed, Azure Blob storage may retain versions or metadata depending on the configuration.
Tested the URL for versioned blobs by appending query parameters like `?comp=versions`
Use Azure Storage Explorer to connect to a blob anonymously.

`?restype=container&comp=list`

![image](https://raw.githubusercontent.com/AmmarM0hammed/Earthlink-Ctf/refs/heads/main/photo_2024-12-28_20-23-19.jpg)

and i found the top secret .zip file thats contains the flag

![image](https://raw.githubusercontent.com/AmmarM0hammed/Earthlink-Ctf/refs/heads/main/photo_2024-12-28_20-39-11.jpg)
 > EL{BLOB_SORAGE_MISS_CONFIG_IS_AMAZING}



## Misc Category

Challange Name : [ textToPixel ]
Poinst : 100
Category : Misc
#### Challenge Description
The challenge description mentioned converting an image to text. The image consisted of only black and white colors. My initial thought was to read the image pixel by pixel, replacing each white pixel with a '0' and each black pixel with a '1'. Then, I planned to convert the resulting binary data into a string using RapidTables Binary to ASCII Converter.

**Step 1**: Converting the Image into a Binary String

To achieve this, I wrote a script to process the image and convert it into a binary string. Here's the script I used:

    from PIL import Image

def process_image(image_path):
    GLOB = "";
    img = Image.open(image_path).convert("L") 
    width, height = img.size

    for y in range(height):
        for x in range(width):
            pixel = img.getpixel((x, y))
            if pixel == 255: 
                GLOB += "1"
            else:
                GLOB += "0"
    return GLOB

image_path = "image.png" 
pixel_data = process_image(image_path)

print(pixel_data)

The output of this step was a binary string that translated into a long text containing only two words: "white" and "black".

**Step 2**: Reconstructing the Image

After obtaining the text, I decided to reconstruct the original image. To do this, I wrote a second script that recreated the image by writing new pixels based on the binary string. At the end of this process, I obtained a QR code. Upon scanning the QR code, voila, I found the flag!

Here is the script I used to convert the long string back into an image:

    from PIL import Image
    
    def draw_image_from_text(input_string, width):
        if not all(word in ["black", "white"] for word in input_string.split()):
            raise ValueError("The input string must only contain the words 'black' and 'white'.")
        words = input_string.split()
        height = len(words) // width + (1 if len(words) % width != 0 else 0)
        img = Image.new("L", (width, height), "white")
        pixels = img.load()
        for i, word in enumerate(words):
            x = i % width
            y = i // width
            color = 0 if word == "black" else 255 
            pixels[x, y] = color
        
        return img
    
    input_string = "[LONG_STRING]".lower()
    width = 35 
    
    image = draw_image_from_text(input_string, width)
    image.show() 
    image.save("output.png")

![image](https://raw.githubusercontent.com/AmmarM0hammed/Earthlink-Ctf/refs/heads/main/photo_2024-12-28_20-39-34.jpg)

Challange Name : [ Mother Of All Math]
Poinst : 100
Category : Misc
#### Challenge Description
The challenge, named Mother of Math, provided an ELF file that turned out to be a game. Upon execution, the game displayed the following:

    Welcome to the Earthlink's Fast Math Quiz! You have only 5 seconds to send the correct answer.
    Note: For long answers, round to 3 decimal points. For power operator, display the first 5 digits after the decimal point.
    Level 1: 3*2
    Your answer:
    Incorrect.
    Quiz is over.

The task was to solve mathematical equations provided by the game within 5 seconds, adhering to specific formatting rules for the answers. The challenge required dynamically solving equations and responding correctly in the given time frame.

**Approach**:

To automate solving the challenge, I developed a Python script using the pwntools library. The script dynamically parsed and solved the equations, ensuring the answers adhered to the rounding requirements. It then interacted with the ELF binary to send the correct answers within the time limit.

    import sys
    from pwn import *
    import re
    import math
    
    def load_elf_game(binary_path):
        print("[INFO] Loading ELF game...")
        return ELF(binary_path)
    
    def solve_equation(equation):
        try:
            equation = equation.replace("^", "**")
            
            equation = equation.replace("sqrt", "math.sqrt")
            
            print(f"[INFO] Solving equation: {equation}")
            result = eval(equation)
            
            if isinstance(result, float):
                if "**" in equation:
                    result = round(result, 5)
                else:
                    result = round(result, 3)
            print(f"[INFO] Computed result: {result}")
            return result
        except Exception as e:
            print(f"[ERROR] Could not solve equation '{equation}': {e}")
            return None
    
    def interact_with_game(game):
        print("[INFO] Starting ELF game process...")
        process = game.process()
        answer = None
    
        while True:
            try:
                prompt = process.recvline(timeout=5).decode('utf-8').strip()
                print(f"[GAME] {prompt}")
                
                if "Level" in prompt:
                    level = int(re.search(r'Level (\d+)', prompt).group(1))
                    print(f"[INFO] Processing Level {level}...")
                    equation = prompt.split(": ")[1]
                    answer = solve_equation(equation)
                elif "Your answer:" in prompt:
                    if answer is not None:
                        print(f"[INFO] Sending answer: {answer}")
                        process.sendline(str(answer).encode())
                    else:
                        print("[ERROR] No answer available to send!")
                        process.sendline(b"0")  
                elif "Incorrect" in prompt or "Correct" in prompt:
                    print(f"[GAME] {prompt}")
                    answer = None  
                elif "Game Over" in prompt:
                    print("[INFO] Game over. Exiting...")
                    break
                else:
                    print("[INFO] Unrecognized prompt. Ignoring...")
    
            except EOFError:
                print("[INFO] End of game detected.")
                break
            except TimeoutError:
                print("[WARNING] Timeout waiting for game response.")
            except Exception as e:
                print(f"[ERROR] An exception occurred: {e}")
                break
    
    def main():
        binary_path = "game.elf"
        print("[INFO] Starting ELF game solver...")
        game = load_elf_game(binary_path)
        interact_with_game(game)
    
    if __name__ == "__main__":
        main()

![image](https://raw.githubusercontent.com/AmmarM0hammed/Earthlink-Ctf/refs/heads/main/photo_2024-12-28_21-03-22.jpg)

![image](https://raw.githubusercontent.com/AmmarM0hammed/Earthlink-Ctf/refs/heads/main/photo_2024-12-28_21-03-25.jpg)

>ELCTF{MaTh_!s_NOT_hARd_AFt3R_A1L}