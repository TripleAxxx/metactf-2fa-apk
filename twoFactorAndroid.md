📌 Challenge Info
Platform: MetaCTF ( https://metaproblems.com/6c6bfeb7a0fc508a452254d65e4c298b/ )   -- site

Challenge: 2FA Android APK (https://metaproblems.com/d84d16e70b19a2069e1e6b06c615e1cc/twoFactorAndroid.apk) --apk file 

Goal: Reverse engineer the OTP mechanism to login as admin and retrieve the flag.

![[Pasted image 20250228044006.png]]

 to login you need admin otp code  an apk file came along with it. I downloaded the APK file and extracted its contents using:
```unzip twoFactorAndroid.apk -d challenge_apk```

 ![[Pasted image 20250228044648.png]]
 then i installed  [jadx](https://github.com/skylot/jadx)  and set it up took alot of time since i cloned it from github and also had to compile it , Then I used JADX to decompile the APK and view the Java code:
 ```
 build/jadx/bin/jadx-gui ~/Downloads/challenge_apk

```
after looking around using the search bar `ctrl+shift+f` to look for hash alogrithms  and encryption types ,  i check the dex file 'classes.dex ' found `MainActivity` tried to  read the java code as much as i can.
 # Understanding the Logic :

the scripts/code uses username and time in millisecond to generate otp code
 - This method takes a string (`name`) as input and performs the following steps:
    1. Converts the string to a character array.
    2. Sums the ASCII values of all characters in the string.
    3. Calculates a time-based value `t` using the current time in milliseconds.
    4. Computes a final value by multiplying the sum of ASCII values by `t` and then by `1337`.
    5. Takes the result modulo `DurationKt.NANOS_IN_MILLIS` (a large constant).
    6. Pads the result to 6 digits with leading zeros.
    7. The `do_math` method is responsible for generating the code. Here’s a step-by-step breakdown of how it works:

1. **Input**: The method takes the user's name (e.g., `"john"`) as input.
2. **ASCII Sum**: It converts the name to a character array and sums the ASCII values of all characters.
    - For `"john"`, the ASCII values are:
        - `j`: 106
        - `o`: 111
        - `h`: 104
        - `n`: 110
    - Sum: `106 + 111 + 104 + 110 = 431`
3. **Time-Based Value**: It calculates a time-based value `t` using the current time in milliseconds.
    - `t = (System.currentTimeMillis() / 1000) / 5`
4. **Final Calculation**: It multiplies the ASCII sum by `t` and then by `1337`.
    - `result = (total * t) * 1337`
5. **Modulo Operation**: It takes the result modulo `DurationKt.NANOS_IN_MILLIS` (a large constant).
6. **Padding**: It pads the result to 6 digits with leading zeros.
---


### **5. Security Implications**

- **Predictability**: The code generation is based on the user's name and the current time, which makes it somewhat predictable.
- **Time-Based Dependency**: The code changes every 5 seconds due to the division by `5` in the time calculation.
- **Potential Vulnerabilities**: If the user's name is user-controlled, it could be manipulated to generate specific codes.

---

### **6. How to EXPLOIT IT **
i create a python script that  generates an otp code  with six digits using username and time 
```
                                                
import time

def do_math(name):
    # Sum the ASCII values of all characters in the name
    total = sum(ord(char) for char in name)

    # Calculate the time-based value
    t = int(time.time()/ 5)

    # Compute the final value
    result = (total * t * 1337) % 1000000

    # Pad the result to 6 digits with leading zeros
    padded_result = str(result).zfill(6)

    return padded_result

# Example usage
if __name__ == "__main__":
    user_name = input("Enter the user's name: ")
    generated_code = do_math(user_name)
    print(f"Generated Code: {generated_code}")





```

![[Pasted image 20250228052859.png]]
got the code now let's try it out
it worked and i got the flag
![[Pasted image 20250228053039.png]]
the otp code last for 5 seconds
