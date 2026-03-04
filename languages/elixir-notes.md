---
description: Basics
---

# Elixir Notes

### Resources

* [https://hexdocs.pm/elixir/introduction.html](https://hexdocs.pm/elixir/introduction.html)

### Conceptual shift

<table><thead><tr><th width="188.30859375">Concept</th><th width="263.4140625">C#</th><th>Elixir</th></tr></thead><tbody><tr><td>Unit of concurrency</td><td><code>Task</code></td><td><code>Process</code></td></tr><tr><td>Scheduling</td><td>OS + .NET runtime</td><td>BEAM VM<br>Fair time slices, no priority/blocks</td></tr><tr><td>State</td><td>Shared (by default)</td><td>Isolated</td></tr><tr><td>Communication</td><td>Method calls, locks</td><td>Messages</td></tr><tr><td>Failure</td><td>Often catastrophic</td><td>Expected &#x26; handled<br>Cashing is normal, restarting is normal, guarding everything is not!</td></tr></tbody></table>

### (Linked) Lists

```elixir
# concat
iex> [1, 2, 3] ++ [4, 5, 6] # [1, 2, 3, 4, 5, 6]

# subtract
iex> [1, true, 2, false, 3, true] -- [true, false] # [1, 2, 3, true]
```

### Tuples

Remember Elixir data structures are immutable and they are **not** like C# tuples! They are stored in contiguous blocks in memory, much like an array list.

```elixir
iex> {:ok, "hello"}
iex> tuple_size({:ok, "hello"})  # 2
iex> put_elem(tuple, 1, "world") # {:ok, "world"}
iex> tuple                       # {:ok, "hello"}

# more real-world usage
user_tuple = {:ok, "Geoffrey", "London"}
{:ok, name, city} = user_tuple
iex> name # "Geoffrey"

# essentially method overloading but no static type checks!
# can use Guards to do similar and order matters
def process_result({:ok, data}) when is_integer(data) and data > 0 do
  IO.puts("I got the data: #{data}")
end
def process_result({:error, reason}) do
  IO.puts("It failed because: #{reason}")
end

# pattern matching
case booking do
  {:ok, id, _ref} -> "ID is #{id}"
  {:error, reason} -> "Error: #{reason}"
end

# variable pinning
# below is the equivalent of: if (get_id() == const_id)
const_id = "123"

case get_id() do
  {:ok, ^const_id} -> "Found the ID!"
  {:ok, other_id} ->  "Found a DIFFERENT ID: #{other_id}"
end
```

### Anonymous functions

Need to use the `.` on the variable that is assigned and is used to avoid ambiguity with module methods.

```elixir
iex> add = fn a, b -> a + b end
iex> add.(1, 2)
```

To use assign a module method to a variable the `&` is used:

```elixir
iex> my_func = &Math.square/1
iex> my_func.(2)
iex> Enum.map([1, 2], my_func)
iex> Enum.map([1, 2], &Math.square/1) # better!
```
