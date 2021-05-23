# [WIP] Python request package

I don't fully know how an HTTP request works in Python. All I know is how to work with Python's `Requests` package. Therefore, I am attempting to understand how an HTTP request works in Pyhton. (I wanted to go down this rabbit hole.)

If you are a Python dev, you have probably used or heard of the `Requests` package. According to the [official doc](https://docs.python-requests.org/en/master/):
> Requests is an elegant and simple HTTP library for Python, built for human beings.

Looking at `Requests` [repo](https://github.com/psf/requests 
), it looks fairly simple. The repo consists of about 20 Python files excluding tests. After skimming throught the code, I've learnted that `Requests` gathers user's input and calls `urllib3.urlopen()` to send an HTTP request.

Ok, now I know that `Requests` isn't the one sending the HTTP request. `Requests` is just an elegant wrapper and `urllib3` is the one sending the HTTP request. 

I cloned [urllib3/urllib3](https://github.com/urllib3/urllib3) and searched for `urlopen` method.

```python
def urlopen(
    self,
    method,
    ...
    **response_kw
):
    """
    Get a connection from the pool and perform an HTTP request. This is the
    lowest level call for making a request, so you'll need to specify all
    the raw details.
    """
```
When I read that urlopen is the lowest level call for making a request, I got pretty excited. But to my disappoint urlopen called Python's [`http`](https://github.com/python/cpython/tree/main/Lib/http) library. So this was not the lowest level call.

Now I cloned [`python/cpython`](https://github.com/python/cpython) to understand the `http` library.

[`http/client.py`](https://github.com/python/cpython/blob/main/Lib/http/client.py) provides this nice diagram.

```
    (null)
      |
      | HTTPConnection()
      v
    Idle
      |
      | putrequest()
      v
    Request-started
      |
      | ( putheader() )*  endheaders()
      v
    Request-sent
      |\_____________________________
      |                              | getresponse() raises
      | response = getresponse()     | ConnectionError
      v                              v
    Unread-response                Idle
    [Response-headers-read]
      |\____________________
      |                     |
      | response.read()     | putrequest()
      v                     v
    Idle                  Req-started-unread-response
                     ______/|
                   /        |
   response.read() |        | ( putheader() )*  endheaders()
                   v        v
       Request-started    Req-sent-unread-response
                            |
                            | response.read()
                            v
                          Request-sent
```
As I understand this diagram, each line of encoded header gets sent to the server by `putheader()` and `endheaders()` checks if all lines of headers are properly sent. Then the `data` gets is sent by `send()`. Yet again, `send()` didn't actually send the request, instead it called `sock.sendall()` from socket library. Apparantly I was doing socket programming this whole time?