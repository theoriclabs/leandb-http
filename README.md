# LeanDB HTTP client

Optional HTTP transport for `LeanDb.Client`, backed by `leanhttp`. The LeanDB
engine itself remains free of an HTTP client or libcurl dependency.

```lean
import LeanDbHttp
import Tickets

open LeanDb

def slaBreachedRemote := client% Tickets.slaBreached

def example : IO (Except DbError (Array (Stored Tickets.Ticket × Stored Tickets.User))) := do
  let client ← HttpClient.connectUrl "https://db.example.com/bases/tickets"
    (fingerprint Tickets.base.specs)
  match client with
  | .error e => return .error e
  | .ok client =>
      let result ← (slaBreachedRemote ⟨1700000000⟩).run client
      client.close
      return result
```

The adapter calls `GET <base>/version` for the fingerprint handshake and
`POST <base>/rpc` thereafter. Every RPC carries `X-LeanDb-Fingerprint`.
