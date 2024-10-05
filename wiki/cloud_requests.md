Cloud Requests Framework (inspired by discord.py) that allows Scratch projects and Python to interact

# Basic usage

**Add Cloud Requests to your Scratch project:**

Download this project file to your computer (click the link to download it): https://github.com/TimMcCool/scratchattach/raw/main/assets/CloudRequests_Template.sb3

Then, go to the Scratch website, create a new project and upload the project file from above.

**How to use with Scratch:**

Copy this code to your Python editor. [How to get your session id](https://github.com/TimMcCool/scratchattach/wiki/Get-your-session-id)

```py
import scratchattach as scratch3

session = scratch3.Session("session_id", username="username") #replace with your session_id and username
conn = session.connect_cloud("project_id") #replace with your project id

client = scratch3.CloudRequests(conn)

@client.request
def ping(): #called when client receives request
    print("Ping request received")
    return "pong" #sends back 'pong' to the Scratch project

@client.event
def on_ready():
    print("Request handler is running")

client.run() #make sure this is ALWAYS at the bottom of your Python file
```

Replace "session_id" and "username" with your data and "project_id" with the id of the project you created on Scratch.
Then, run the code.

Now go to the Scratch project. In the `Cloud Requests` sprite, you will find this block:

![image](https://github.com/TimMcCool/scratchattach/blob/main/wiki/images/cr_tut_block.png)

When active, it sends a "ping" request to the Python client. This will call the `ping()` function. The data returned by the function will be sent back to the project.
Try it out by clicking the block!

![image](https://github.com/TimMcCool/scratchattach/blob/main/wiki/images/cr_tut_restult.png)

**How to use with TurboWarp:**

```python
import scratchattach as scratch3

conn = scratch3.TwCloudConnection(project_id="project_id", purpose="your use case (optional)", contact="your Scratch account or other contact info (optional)") #replace with your project id
client = scratch3.TwCloudRequests(conn)

...
```

# Examples

**Example 1: Script that loads your message count**

Scratch code:

![image](https://github.com/TimMcCool/scratchattach/blob/main/wiki/images/cr_tut_example1.png)

Python code (add this to the code from above, but make sure `client.run()` stays at the bottom of the file):

```python
@client.request
def message_count(argument1):
    print(f"Message count requested for user {argument1}")
    user = scratch3.get_user(argument1)
    return user.message_count()
```

The arguments you specify in the Scratch code are given to the Python function.

**Example 2: Script that loads someone's stats**

Scratch code:

![image](https://github.com/TimMcCool/scratchattach/blob/main/wiki/images/cr_tut_example3.png)

Python code:

```python
@client.request
def foo(argument1):
    print(f"Data requested for user {argument1}")
    user = scratch3.get_user(argument1)
    stats = user.stats()

    return_data = []
    return_data.append(f"Total loves: {stats['loves']}")
    return_data.append(f"Total favorites: {stats['favorites']}")
    return_data.append(f"Total views: {stats['views']}")

    return return_data
```

# Basic features

- **No length limitation** for the request or the returned data! (If it is too long for one cloud variable, it will be split into multiple cloud variables)
- Cloud Requests can handle **multiple requests sent at the same time**
- Requests can also **return lists,** these will be decoded as list in the Scratch project
- You can freely choose the name of your requests
- Requests that only return numbers won't be encoded (-> 50% faster!)

# Advanced features

**no_packet_loss mode:** (new in v1.1.9)

Disabled by default. When enabled, the request handler will reconnect to the cloud websocket after every single request to make sure there is no packet loss. This however causes the request handler to respond slower. How to enable no_packet_loss:
```py
client.run(no_packet_loss=True)
```

**Get the request metadata:**

In your requests, you can use these functions (Warning: If you have requests that are running in threads they may not work):
```py
client.get_requester() #Returns the name of the user who sent the request
client.get_timestamp() #Returns the timestamp of when the request was sent (in milliseconds since 1970)
client.get_exact_timestamp() #Returns the exact timestamp of when the request was sent (fetches it from the clouddata logs). New in v1.2.6
```

**Run cloud requests in a thread:**

By default, this is disabled. How to enable:
```py
client.run(thread=True, daemon=False) #set daemon to True to run the request handler in a daemon thread
```
If enabled, you can put code below the client.run function, and you can use `client.stop()` to stop a running cloud requests instance.

**Change what "FROM_HOST_" cloud vars are used:**
```py
client = scratch3.CloudRequests(conn, used_cloud_vars=["1","2","3","4","5",...])
```
This allows you to set what "FROM_HOST_" cloud variables the Python script uses to send data back to your project. You can remove unused "FROM_HOST_" cloud variables from the Scratch project.

**Ignore exceptions:**

By default, the request handler will ignore exceptions occuring in your requests. You can also make it raise these exceptions instead:
```py
client = scratch3.CloudRequests(conn, ignore_exceptions=False)
```

**Send more than two arguments:**

The seperator used to join the different arguments is "&". To send more than three arguments from Scratch, join them using "&".

**Change packet length and clouddata log URL** (Warning: Only change this if you know what you are doing):

```py
client = scratch3.CloudRequests(conn, _log_url="https://clouddata.scratch.mit.edu/logs", _packet_length=220)
```

**Change data source** (not recommended)

```py
client.run(data_from_websocket=False) #to fetch data from the clouddata logs instead
```

# Advanced requests
(new in v1.0.0)

**Add arguments to the decorator:**

Above requests, you put the decorator `@client.request`.
You can use this decorator to customize your requests!

*Run request in thread*

Put this decorator above a request to run it in a thread (makes it possible to run multiple request simultaneously):

Warning: This feature is currently very unoptimized and will be improved soon.

```py
@client.request(thread=True)
```

*Disable request:*
Put this decorator above a request to disable it:
```py
@client.request(enabled=False)
```

*Overwrite the request name:*
Put this decorator above a request to overwrite its name with something else:
```py
@client.request(name="new_name")
```

**Manually add, edit and remove requests:**

*Add requests:*
To add requests, you can also use this method:

```py
client.add_request(function, thread=False, enabled=True, name="request_name")
```

*Edit / Remove requests:*
```py
client.edit_request("request_name", thread=False, enabled=True, new_name="request_name", new_function=new_function) #The keyword arguments are optional and can be removed if they are not needed
client.remove_request("request_name")
```

# Advanced events

Events will be called when specific things happen.
There are more events than just `on_ready`!

*Called after the client connected to the cloud:*
```py
@client.event
def on_ready():
    print("Request handler is ready")
```

*Called every time the client receives any request:*
```py
@client.event
def on_request(request):
    print("Received request", request.name, request.requester, request.arguments, request.timestamp, request.id)
```

*Called when the client receives an unknown / undefined request:*
```py
@client.event
def on_unknown_request(request):
    print("Received unknown request", request.name, request.requester, request.arguments, request.timestamp, request.id)
```

*Called when an error occurs in a request:*
```py
@client.event
def on_error(request, e):
    print("Request: ", request.name, request.requester, request.arguments, request.timestamp, request.id)
    print("Error that occured: ", e)
```

*Called when the client receives a disabled request:*
```py
@client.event
def on_disabled_request(request):
    print("Received disabled request", request.name, request.requester, request.arguments, request.timestamp, request.id)
```
