---
description: Basics
---

# Elixir Notes

### Conceptual shift

<table><thead><tr><th width="188.30859375">Concept</th><th width="263.4140625">C#</th><th>Elixir</th></tr></thead><tbody><tr><td>Unit of concurrency</td><td><code>Task</code></td><td><code>Process</code></td></tr><tr><td>Scheduling</td><td>OS + .NET runtime</td><td>BEAM VM<br>Fair time slices, no priority/blocks</td></tr><tr><td>State</td><td>Shared (by default)</td><td>Isolated</td></tr><tr><td>Communication</td><td>Method calls, locks</td><td>Messages</td></tr><tr><td>Failure</td><td>Often catastrophic</td><td>Expected &#x26; handled<br>Cashing is normal, restarting is normal, guarding everything is not!</td></tr></tbody></table>

Let's take the example of an "expected failure":

```elixir
def fetch_user(id) do
  case Repo.get(User, id) do
    nil -> {:error, :not_found}
    user -> {:ok, user}
  end
end
```

This is like `Result<T>` / `OneOf` style in C#.

Now for "unexpected failures:

```elixir
def fetch_user!(id) do
  Repo.get!(User, id)   # will raise if not found
end
```

The supervisor will decide whether/how to restart it. For C#, it's as if each request handler has its own tiny isolated runtime and automatic restart policy.
