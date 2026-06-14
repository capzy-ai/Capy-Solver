# Parameters reference — Capy Puzzle Captcha

Every field you can pass to `POST /createTask` for this task type.

## Envelope

```json
{
  "clientKey": "capzy_xxxxxxxxxxxxxxxxxxxxxxxx",
  "task": { ... }
}
```

| Field        | Required | Notes                                                       |
|--------------|:--------:|-------------------------------------------------------------|
| `clientKey`  | yes      | Your Capzy API key. Starts with `capzy_`. Find it at [capzy.ai/dashboard/api-keys](https://capzy.ai/dashboard/api-keys). |
| `task`       | yes      | The task object — see below.                                |

## Task object

### Required + optional fields

| Field | Type | Required | Notes |
|-------|------|:--------:|-------|
| `type` | `string` | yes | CapyTaskProxyLess or CapyTask |
| `websiteURL` | `string` | yes | Full URL of the page |
| `websiteKey` | `string` | yes | Capy sitekey (starts with `PUZZLE_`) |
| `apiServer` | `string` | no | Custom Capy widget host. Default `https://jp.api.capy.me`. Pass this when the target site serves the widget from its own CDN (white-label Capy deployments) — e.g. `https://puzzleauth.captchasolutionweb.com`. Get the value from the `<script src="…/puzzle/get_js/?k=…">` tag on the target page. Aliases (any of these are accepted, same behavior): `api_server`, `capyApiServerSubdomain`. |


### Proxy fields (only for `CapyTask`)

| Field | Type | Required | Notes |
|-------|------|:--------:|-------|
| `proxyType` | `string` | no | http | https | socks4 | socks5 |
| `proxyAddress` | `string` | no | IP or hostname of your proxy |
| `proxyPort` | `integer` | no | Port number of your proxy |
| `proxyLogin` | `string` | no | Optional — omit if your proxy doesn't require auth |
| `proxyPassword` | `string` | no | Optional — omit if your proxy doesn't require auth |


## Response

### `POST /createTask` success

```json
{
  "errorId": 0,
  "taskId":  "12345"
}
```

### `POST /getTaskResult` while processing

```json
{
  "errorId": 0,
  "status":  "processing"
}
```

### `POST /getTaskResult` when ready

```json
{
  "errorId":  0,
  "status":   "ready",
  "solution": { ... }
}
```

The `solution` object contains:

| Field | Type | Notes |
|-------|------|-------|
| `captchakey` | `string` | Challenge identifier |
| `challengekey` | `string` | Per-session challenge value |
| `answer` | `string` | Hex-encoded solved puzzle answer |

### How to use the solution

Submit `captchakey`, `challengekey`, and `answer` to the target site's form exactly as Capzy returns them.

### Error

```json
{
  "errorId":          1,
  "errorCode":        "ERROR_KEY_DOES_NOT_EXIST",
  "errorDescription": "Invalid API key"
}
```

`errorId` is `0` on success, `1` on any error. The `errorCode` is the
stable machine-readable identifier. Common codes:

- `ERROR_KEY_DOES_NOT_EXIST` — bad API key
- `ERROR_NO_BALANCE` — account balance below the cost of this task
- `ERROR_INVALID_PARAMS` — missing required field or malformed value
- `ERROR_MAX_TASKS_REACHED` — concurrent in-flight cap reached (default 30)
- `ERROR_RATE_LIMITED` — too many createTask calls per second
- `ERROR_TIMEOUT` — solve took longer than the cap (auto-refunded)
- `ERROR_CAPTCHA_UNSOLVABLE` — solver gave up (auto-refunded)
- `ERROR_CAPTCHA_INVALID_SITEKEY` — `websiteKey` is invalid, expired, deactivated, or registered against a different host than the default Capy CDN. Pre-flight probe catches this in <1 s; the error message echoes the host we checked. If the target site uses a white-label Capy deployment, pass `apiServer` (see above) — verifying the script src on the target page is the fastest way to find the right value.

## Naming conventions

Field names are camelCase on the wire (`websiteURL`, `websiteKey`,
`proxyAddress`). Stick to that exactly when you build the JSON.
