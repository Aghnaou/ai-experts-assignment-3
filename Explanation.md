## What was the bug?

The code wasn't checking if dictionary tokens were expired. When `oauth2_token` was a dict instead of an `OAuth2Token` object, the refresh logic would skip over it completely. This meant expired tokens stored as dicts never got refreshed, so no Authorization header was added to API requests.

## Why did it happen?

The original refresh condition looked like this:

```python
if not self.oauth2_token or (
    isinstance(self.oauth2_token, OAuth2Token) and self.oauth2_token.expired
):
    self.refresh_oauth2()
```

This only handles two cases:
1. Token is None/missing
2. Token is an OAuth2Token object AND it's expired

But when the token is a dictionary, the `isinstance` check fails, so the `.expired` property is never accessed. The code just assumes the dict token is fine and moves on without refreshing.

## Why does my fix solve it?

I added a third condition that specifically checks for dict tokens:

```python
isinstance(self.oauth2_token, dict) and 
int(datetime.now(tz=timezone.utc).timestamp()) >= self.oauth2_token.get("expires_at", 0)
```

This checks if the token is a dict, then compares the current timestamp against the `expires_at` value stored in the dict. If the current time is past the expiration, it triggers a refresh just like it would for an expired OAuth2Token object.