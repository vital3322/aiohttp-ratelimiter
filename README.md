<a href="https://jgltechnologies.com/discord">
<img src="https://discord.com/api/guilds/844418702430175272/embed.png">
</a>

# aiohttp-ratelimiter

aiohttp-ratelimiter is a rate limiter for the aiohttp.web framework.
This is a new library and we are always looking for people to contribute. If you see something wrong with the code or want to add a feature, please create a pull request 
on <a href="https://jgltechnologies.com/aiohttplimiter">our github</a>.


Install from git
```
python -m pip install git+https://github.com/Nebulizer1213/aiohttp-ratelimiter
```

Install from pypi
```
python -m pip install aiohttp-ratelimiter
```

<br>


Example

```python
from aiohttp import web
from aiohttplimiter import default_keyfunc, Limiter

app = web.Application()
routes = web.RouteTableDef()

limiter = Limiter(keyfunc=default_keyfunc)

@routes.get("/")
# This endpoint can only be requested 1 time per second per IP address
@limiter.limit("1/1")
async def home(request):
    return web.Response(text="test")

app.add_routes(routes)
web.run_app(app)
```

<br>

You can exempt an IP from rate limiting using the exempt_ips kwarg.

```python
from aiohttplimiter import Limiter, default_keyfunc
from aiohttp import web

app = web.Application()
routes = web.RouteTableDef()

# 192.168.1.245 is exempt from rate limiting.
# Keep in mind that exempt_ips takes a set not a list.
limiter = Limiter(keyfunc=default_keyfunc, exempt_ips={"192.168.1.245"})

@routes.get("/")
@limiter.limit("1/1")
async def test(request):
    return web.Response(text="test")

app.add_routes(routes)
web.run_app(app)
```

<br>

You can create your own error handler by using the error_handler kwarg.

```python
from aiohttplimiter import Allow, RateLimitExceeded, Limiter, default_keyfunc
from aiohttp import web

def handler(request: web.Request, exc: RateLimitExceeded):
    # If for some reason you want to allow the request, return aiohttplimitertest.Allow().
    if some_condition:
        return Allow()
    return web.Response(text="Too many requests", status=429)

limiter = Limiter(keyfunc=default_keyfunc, error_handler=handler)
```

<br>

If multiple paths use one handler like this:
```python
@routes.get("/")
@routes.get("/home")
@limiter.limit("5/1")
def home(request):
    return web.Response(text="Hello")
```

<br>

Then they will have separate rate limits. To prevent this use the path_id kwarg.

```python
@routes.get("/")
@routes.get("/home")
@limiter.limit("5/1", path_id="home")
def home(request):
    return web.Response(text="Hello")
```

<br>

If you want to use Redis instead, use the RedisLimiter class. 

```python
from aiohttplimiter import RedisLimiter, default_keyfunc


limiter = RedisLimiter(keyfunc=default_keyfunc, uri="redis://:password@host:port")
```
<p style="color: red;">
RedisLimiter is still being tested and might not be stable for production.
</p>



