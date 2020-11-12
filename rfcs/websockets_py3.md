# RFC : Add Python3 WebSockets dependency for WebDriver Bidi tests

## Summary

Add Py3-only [WebSockets client](https://github.com/aaugustin/websockets) that uses asyncio for WebDriver Bidi tests. 

## Details

### Background

[WebDriver Bidi](https://w3c.github.io/webdriver-bidi/) builds and extends on [WebDriver](https://w3c.github.io/webdriver/).
WebDriver Bidi uses WebSockets for bidirectional communication between remote end and local end.
Existing pywebsocket3 library only serves as a websocket server. Therefore, a new WebSockets client dependency is needed in order to connect to WebDriver for Bidi communication.  

### Proposal

Add Py3-only WebSockets library linked in Summary.

Add [pytest-asyncio](https://github.com/pytest-dev/pytest-asyncio) for convenient test marker.
Note pytest-asyncio depends on later versions of pytest and pluggy so they also need to upgrade to compatible versions.

### Example usage

The [patch](https://github.com/web-platform-tests/wpt/pull/26510) 
1. Adds a simple test to verify the new capability that would enable websocket connection for Bidi communication.
1. Upgrades existing tools/third_party dependencies which are depended by the pytest-asyncio which provides convenient markers such as
"@pytest.mark.asyncio" for writing tests.
1. Makes use of the async/await syntax.
1. Adds a BidiSession class in tools/webdriver/webdriver/client.py to be used
in test fixture.
1. Adds tests to verify the behavior of BidiSession across tests.

A couple issues not solved in the patch:

1. When upgrading pytest to 6.1.1 in example code, there was an error from tools/third_party/pytest/src/_pytest/assertion/rewrite.py
about missing _pytest._version module.
I commented out the import and hardcoded the version to 6.1.1.
1. I installed the WebSockets and pytest-asyncio dependencies locally instead of including them in tools/third_party library.

## Risks

The current plan for python migration(I believe) is Py3-first by the end of the year and Py3-only Feb 2021.
The patch is Py3-only and there is a gap of 3 months from now and then.
Do we want to wait until then to add the dependencies?
Or do we want to be able to exclude these dependencies for Py2?