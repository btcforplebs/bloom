# Bloom Troubleshooting Guide

## Common Errors and Solutions

### 401 Unauthorized Errors

#### Symptoms
- `Failed to load resource: the server responded with a status of 401`
- Errors when uploading files to Blossom servers
- Errors when viewing/downloading blobs

#### Causes and Solutions

**1. Server Requires Auth But Not Configured**

Your Blossom server (`haven.btcforplebs.com`) requires authentication. Make sure:

1. **Check Server Configuration**:
   - Go to Settings → Servers
   - Find your server in the list
   - Ensure the "Requires Auth" checkbox is checked
   - If not, check it and save

2. **Verify Signer Connection**:
   - Make sure you're logged in with a Nostr signer (Alby, nos2x, etc.)
   - Check the connection status in the top right
   - If disconnected, click "Connect" and authorize with your signer

**2. Authorization Event Issues**

The server might be rejecting your auth events. Check the browser console for:

```
Creating auth event: { kind: "upload", created_at: ..., expiration: ..., hash: "..." }
```

Common problems:
- **Clock Skew**: If your computer's clock is off, the `created_at` might appear in the future to the server
- **Expired Event**: Events expire after 5 minutes (300 seconds) by default
- **Hash Mismatch**: The `x` tag must match the file's SHA-256 hash exactly

**3. Server Allowlist**

Some Blossom servers only accept uploads from specific pubkeys:
- Contact the server admin to add your pubkey: `${YOUR_PUBKEY}`
- Or use a different server that allows public uploads

**4. Missing Server Configuration**

If you added the server manually, ensure all fields are correct:
- **URL**: Must start with `https://` or `http://`
- **Type**: Should be "blossom" (not "nip96" or "satellite")
- **Requires Auth**: Must be checked if the server requires authentication

### "BUG: No filters to merge!" Errors

#### Symptoms
```
BUG: No filters to merge! – Map (0)
compileFilters (ndk-DYtfbXzt.js:1:31308)
```

#### Solution

This is an NDK library bug that occurs when trying to fetch events without any connected relays.

**Fixed in latest version** - The code now checks for relay availability before fetching.

If you still see this error:
1. Go to Settings → Relays
2. Add at least one relay (e.g., `wss://relay.damus.io`)
3. Ensure at least one relay shows "Connected" status
4. Refresh the page

### WebSocket Connection Failures

#### Symptoms
```
WebSocket connection to 'wss://haven.btcforplebs.com/' failed: There was a bad response from the server.
```

#### Causes

1. **Not a WebSocket Endpoint**: Blossom servers use HTTP/HTTPS, not WebSocket
   - This error appears when NDK tries to connect to a Blossom server as if it were a relay
   - **This is normal and can be ignored** - Bloom uses HTTP for Blossom operations

2. **Relay Connection Issues**: If the error is for an actual relay:
   - The relay might be down or unreachable
   - Your network might be blocking WebSocket connections
   - Try using a different relay

### Upload Debugging

When uploading files, you should see console logs like:

```
Creating auth event: {
  kind: "upload",
  created_at: 1234567890,
  expiration: 1234568190,
  hash: "75462f4dece4fbde...",
  tags: "t, x, expiration, server, url, name, size, type"
}

Uploading to Blossom server: {
  url: "https://haven.btcforplebs.com/upload",
  fileName: "example.jpg",
  size: 123456,
  type: "image/jpeg",
  hash: "75462f4dece4fbde...",
  requiresAuth: true,
  skipSizeTag: false
}
```

If you see `requiresAuth: false` but get 401 errors:
1. The server configuration is wrong
2. Edit the server and check "Requires Auth"
3. Try uploading again

### Blob Download Debugging

When viewing blobs that require auth, you should see:

```
Building GET authorization for blob: {
  sha: "75462f4dece4fbde...",
  serverUrl: "https://haven.btcforplebs.com",
  urlPath: "/75462f4dece4fbde54a535cfa09eb0d329bda090a9c2f9ed6b5f9d1d2fb6c15b",
  fullUrl: "https://haven.btcforplebs.com/75462f4dece4fbde54a535cfa09eb0d329bda090a9c2f9ed6b5f9d1d2fb6c15b"
}
```

## How to Enable Debug Logging

Open browser console (F12) and run:

```javascript
// Enable all Bloom debug logs
localStorage.setItem('debug', 'bloom:*')

// Or specific subsystems
localStorage.setItem('debug', 'bloom:upload,bloom:auth,bloom:nip94')
```

Then refresh the page.

## Blossom Authorization Spec

Bloom follows the Blossom specification for authorization (BUD-01, BUD-02):

### Upload Authorization Event (kind 24242)
```json
{
  "kind": 24242,
  "content": "",
  "created_at": <current_unix_timestamp>,
  "tags": [
    ["t", "upload"],
    ["x", "<file_sha256_hash>"],
    ["expiration", "<unix_timestamp_5min_future>"],
    ["server", "https://server.com"],
    ["url", "/upload"],
    ["name", "filename.jpg"],
    ["size", "123456"],
    ["type", "image/jpeg"]
  ]
}
```

### GET Authorization Event (kind 24242)
```json
{
  "kind": 24242,
  "content": "",
  "created_at": <current_unix_timestamp>,
  "tags": [
    ["t", "get"],
    ["x", "<file_sha256_hash>"],
    ["expiration", "<unix_timestamp_2min_future>"]
  ]
}
```

The event is then:
1. Signed by your Nostr signer
2. Base64 encoded
3. Sent in the `Authorization: Nostr <base64_event>` header

## Getting Help

If you're still experiencing issues:

1. **Check the browser console** for detailed error messages
2. **Verify your server settings** in Settings → Servers
3. **Test with a public server** (e.g., `https://blossom.primal.net`)
4. **Check your clock** - Make sure your system time is accurate
5. **Try a different browser** - Rules out browser-specific issues

### Reporting Bugs

When reporting issues, please include:
- Browser and version
- Error messages from console
- Steps to reproduce
- Screenshot of Settings → Servers page
- Your Nostr pubkey (npub)
