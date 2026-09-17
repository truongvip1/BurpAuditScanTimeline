# Active Scan Timeline Viewer

`Active Scan Timeline Viewer` is a Burp Suite Professional extension for reviewing **all HTTP traffic emitted by the native Active Scanner**. It is not restricted to findings: every Scanner request is listed, along with its response when received.

## Interface

- **All scanner traffic** is an always-on tab that contains every request from Burp Scanner after the extension loads, including scans that produce no findings.
- Tabs **1**, **2**, **3**, and so on are created immediately when you select **Active scan with Timeline** from a request's right-click menu. Each contains the request/response list for that scan.
- Each tab uses an Intruder-like list with request, inferred payload, inferred insertion point, status code, response received, error, timeout, and response length.
- Selecting a row displays the raw request and response underneath the list. CSV export and copy-selected-row are also available.
- Select a numbered tab and click **Rename selected tab** to give the scan a meaningful name.

## Start an Active Scan with Timeline

1. In Proxy history, Target, Repeater, or another HTTP-message view, right-click the request.
2. Open Burp's **Extensions** submenu and choose **Active scan with Timeline**.
3. The extension creates the next numbered tab before starting the audit, then starts an Active Scan and routes all subsequent Scanner traffic to that tab.

Choosing **Active scan with Timeline** again creates the next numbered tab and routes new Scanner traffic to it. Run one custom scan at a time: Burp does not publish the native per-request task ID needed to separate overlapping audits.

The action calls Montoya `Scanner.startAudit`, so Burp creates a real Scanner audit task that is visible in the Dashboard. The selected tab's status bar and tooltip show the scanner's task message plus its request, insertion-point, finding, and error counts. Burp's public extension API does not provide a definitive task-completed callback, but these counters make progress visible. All individual requests are retained in **All scanner traffic**.

The **Active scan with Timeline** action uses Montoya's built-in `LEGACY_ACTIVE_AUDIT_CHECKS` configuration. It does not read or reuse a custom scan configuration selected in Burp's native scan wizard, because that configuration is not exposed by the public API.

## What cannot be obtained from the public API

- Burp does not expose the native Scanner's private audit-task ID, exact payload, exact insertion-point metadata, or individual network exception to extensions.
- `Payload` and `Insert point` are marked as **best-effort inferences** by comparing requests with the scan's base request.
- When no response callback arrives after 120 seconds, the row is provisionally marked `Likely` timeout with `Connection error or timeout`. A later response replaces that state.

The request, response, status, response time, and response length are recorded directly from Montoya HTTP callbacks.

## Build

Requirements: JDK 17+ and PowerShell. Maven/Gradle are not required.

```powershell
Set-Location D:\ExtBurp\ScanViewer
.\build.ps1
```

The script downloads the Montoya API from Maven Central on first run and creates:

`target\active-scan-timeline-viewer.jar`

## Install

1. In Burp Suite Professional, go to **Extensions** → **Installed** → **Add**.
2. Select `target\active-scan-timeline-viewer.jar` as a Java extension.
3. Use the **Audit Timeline** suite tab while running native Active Scans.
