# Anna's Archive waiting-room fix

The internal browser used to return the waiting-room HTML and close its incognito
session. The downloader slept and fetched the original URL again, creating another
browser session and often restarting the timer. Keep the same tab alive until the
site's own JavaScript countdown and automatic navigation finish. Do not skip the
site's wait or manually reload it. HTTP 200 and cached-cookie waiting rooms also
enter the browser. External bypasser behavior is unchanged.

Waiting is cancellable and bounded to 300 seconds inside the existing browser
watchdog. A stuck queue ends the current solve instead of opening another browser.
Transient navigation errors, a zero timer, and passive protection after a reload
are not treated as completed download pages.

## Validation

`uv run --extra browser pytest -n 0 tests/bypass tests/download/test_http_aa_redirects.py tests/download/test_ddg_check_probe_handoff.py tests/download/test_ddg_handshake_cookies.py`

The live waiting-room diagnostic kept one browser open and observed the timer
reach zero, followed by a real download link. Download-file host availability is
separate from successfully finishing the waiting room.

## Stable deployment overlay

Build this directory with `docker build --pull --build-arg FORK_REVISION=<commit>
-t ebooks/shelfmark:broglea-waitlist .`. The base defaults to upstream `latest`
(stable); override `BASE_IMAGE` with a digest for a reproducible deployment.
Only the three changed Python files are patched, avoiding unrelated main-branch
changes. Upstream metadata still identifies the base; the `io.broglea.*` labels
identify the fork fix.

Weekly server updates pull a new upstream base and rebuild this overlay before
stopping any applications. Patch context must match with zero fuzz. An incompatible
upstream release fails the build and leaves the running services in place; inspect
the update service journal and rebase the fix before retrying. This intentionally
does not silently discard the fix or overwrite changed upstream code.

Regenerate the overlay after changing the fix (stage any new Python files first):

```sh
git diff 46d21ca -- shelfmark/bypass/internal_bypasser.py shelfmark/bypass/waiting_room.py shelfmark/download/http.py > deploy/waitlist/waiting-room.patch
```
