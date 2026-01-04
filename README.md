# HTB-Command_Injections

## Table of Contents
1. [Exploitation](#exploitation)
    1. [Detection](#detection)
    2. [Injecting Commands](#injecting-commands)
    3. [Other Injection Operators](#other-injection-operators)

## Tools/Useful Links

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


