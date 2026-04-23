# Textual Workers Reference

Complete reference for async workers and threading in Textual.

## Why Workers

Long-running or blocking operations (network requests, file I/O, CPU-heavy tasks) must not run in event handlers directly, as they block the UI. Workers run these operations concurrently.

## The `@work` Decorator

The simplest way to create a worker:

```python
from textual import work
from textual.app import App

class MyApp(App):
    @work
    async def fetch_data(self, url: str) -> None:
        """Runs as an async worker."""
        response = await httpx.AsyncClient().get(url)
        self.notify(f"Got {len(response.content)} bytes")

    def on_mount(self) -> None:
        self.fetch_data("https://api.example.com/data")
```

### @work Options

```python
@work(
    exclusive=False,     # Cancel previous workers in same group
    group="default",     # Worker group name
    exit_on_error=True,  # Exit app on unhandled error
    thread=False,        # Run in thread instead of async
    name="",             # Worker name for debugging
    description="",      # Longer description
)
```

### Exclusive Workers

Only one worker runs at a time in its group. Starting a new one cancels the previous:

```python
class SearchApp(App):
    @work(exclusive=True, group="search")
    async def search(self, query: str) -> None:
        """Previous search is cancelled when a new one starts."""
        results = await self.api.search(query)
        self.display_results(results)

    def on_input_changed(self, event: Input.Changed) -> None:
        self.search(event.value)  # Each keystroke starts a new search
```

## run_worker Method

Alternative to `@work` for more control:

```python
class MyApp(App):
    def on_mount(self) -> None:
        worker = self.run_worker(
            self.do_work(),
            name="background-task",
            group="tasks",
            exclusive=True,
            exit_on_error=True,
            thread=False,
        )

    async def do_work(self) -> str:
        await asyncio.sleep(2)
        return "done"
```

## Thread Workers

For blocking (non-async) operations that can't use `await`:

```python
import httpx
from textual import work

class MyApp(App):
    @work(thread=True)
    def fetch_sync(self, url: str) -> None:
        """Runs in a thread. Use thread-safe operations only."""
        response = httpx.get(url)  # Blocking call
        self.call_from_thread(self.notify, f"Got {len(response.content)} bytes")

    @work(thread=True)
    def load_file(self, path: str) -> str:
        """Read a file in a thread."""
        with open(path) as f:
            data = f.read()
        self.call_from_thread(self.update_content, data)
        return data
```

### Thread Safety

Thread workers run outside the main async loop. To safely interact with the UI:

```python
# Post a message from a thread
self.call_from_thread(self.post_message, MyMessage(data))

# Call any method safely
self.call_from_thread(self.notify, "Done!")

# Update a widget from a thread
self.call_from_thread(widget.update, "New content")
```

**Never** directly modify widgets or post messages from a thread worker without `call_from_thread`.

## Checking Cancellation

Workers can be cancelled. Check `is_cancelled` for long-running operations:

```python
from textual.worker import get_current_worker

class MyApp(App):
    @work(exclusive=True)
    async def process_items(self, items: list) -> None:
        worker = get_current_worker()
        for item in items:
            if worker.is_cancelled:
                return  # Stop processing
            await self.process_one(item)
```

## Worker States

| State | Description |
|-------|-------------|
| `PENDING` | Created but not yet running |
| `RUNNING` | Currently executing |
| `CANCELLED` | Was cancelled before completion |
| `ERROR` | Finished with an error |
| `SUCCESS` | Finished successfully |

## Worker Events

Workers emit `Worker.StateChanged` messages:

```python
from textual.worker import Worker

class MyApp(App):
    def on_worker_state_changed(self, event: Worker.StateChanged) -> None:
        if event.state == Worker.WorkerState.SUCCESS:
            self.notify(f"Worker {event.worker.name} completed")
        elif event.state == Worker.WorkerState.ERROR:
            self.notify(f"Worker failed: {event.worker.error}", severity="error")
```

## Worker Properties

```python
worker = self.run_worker(coro)

worker.state          # Current WorkerState
worker.is_running     # True if running
worker.is_finished    # True if done (success, error, or cancelled)
worker.is_cancelled   # True if cancelled
worker.result         # Return value (after SUCCESS)
worker.error          # Exception (after ERROR)
worker.name           # Worker name
worker.group          # Worker group
worker.node           # Widget/App that created the worker
worker.progress       # Progress percentage (0-100)
```

## Cancelling Workers

```python
# Cancel a specific worker
worker.cancel()

# Cancel all workers in the default group
self.workers.cancel_group(self, "default")

# Cancel all workers for this node
self.workers.cancel_node(self)
```

## Common Patterns

### Loading Data on Mount

```python
class DataView(Widget):
    @work
    async def load_data(self) -> None:
        self.loading = True
        try:
            data = await fetch_data()
            table = self.query_one(DataTable)
            table.add_rows(data)
        finally:
            self.loading = False

    def on_mount(self) -> None:
        self.load_data()
```

### Debounced Search

```python
class SearchApp(App):
    @work(exclusive=True, group="search")
    async def search(self, query: str) -> None:
        await asyncio.sleep(0.3)  # Debounce
        worker = get_current_worker()
        if worker.is_cancelled:
            return
        results = await self.api.search(query)
        self.display_results(results)

    def on_input_changed(self, event: Input.Changed) -> None:
        self.search(event.value)
```

### Progress Tracking

```python
class MyApp(App):
    @work
    async def process(self, items: list) -> None:
        bar = self.query_one(ProgressBar)
        bar.update(total=len(items))
        for i, item in enumerate(items):
            await self.process_item(item)
            bar.advance(1)
```

### Parallel Workers

```python
class MyApp(App):
    @work(group="downloads")
    async def download(self, url: str) -> None:
        """Multiple downloads can run in parallel (not exclusive)."""
        data = await fetch(url)
        self.save(url, data)

    def on_mount(self) -> None:
        for url in self.urls:
            self.download(url)  # Each creates a separate worker
```
