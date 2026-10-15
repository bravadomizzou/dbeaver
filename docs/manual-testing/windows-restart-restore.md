# Manual test: DBeaver reopens after Windows restart

## What the feature does

On Windows, DBeaver registers itself with the Windows application restart API at startup.
If DBeaver is still running when Windows restarts, shuts down, or installs updates,
Windows launches DBeaver again after the user signs back in.
DBeaver then restores its windows, layout, and open editors from the saved workbench state.

Crash and hang restarts are not registered. DBeaver does not relaunch itself after a crash.

## Prerequisites

1. Use Windows 10 version 1709 or later, or Windows 11.
2. Turn on the Windows setting that restarts apps:
   **Settings > Accounts > Sign-in options > "Automatically save my restartable apps and restart them when I sign back in"**.
3. Build DBeaver CE from this branch:
   ```
   mvn package -f product/aggregate/pom.xml -T1C -Pproduct-dbeaver-ce,product-dbeaver-eclipse-ce
   ```
4. Run DBeaver from the built `dbeaver.exe`. Do not run it from an IDE launch configuration.
   Windows relaunches the process executable, so it must be `dbeaver.exe`, not `java.exe` or `javaw.exe`.
5. DBeaver skips its own "Confirm exit" and "cancel running tasks" prompts when Windows ends the session.
   Prompts for unsaved editors and open transactions can still appear.

## Test 1: Restart restores DBeaver

1. Start DBeaver and connect to any database, for example the sample database.
2. Open two SQL editors and one table editor.
3. Resize the main window and move it to a non-default position. Or maximize it.
4. Rearrange a view, for example move or minimize the Project view.
5. Wait at least 60 seconds. Windows only restarts apps that have run for at least one minute.
6. Restart Windows with **Start > Power > Restart**. Leave DBeaver open.
7. Sign back in.

**Expected:** DBeaver starts on its own within about a minute of signing in.
The window size, position, and maximized state match step 3.
The view layout matches step 4.
The SQL editors and table editor from step 2 are open again.

## Test 2: Shut down and power on

1. Repeat steps 1 to 5 of Test 1.
2. Choose **Start > Power > Shut down**, then power the machine back on and sign in.

**Expected:** Same as Test 1.

## Test 3: DBeaver closed before restart

1. Start DBeaver, wait 60 seconds, then close it with **File > Exit**.
2. Restart Windows.

**Expected:** DBeaver does not start after sign-in.

## Test 4: Windows setting turned off

1. Turn off the "restart apps" setting from the prerequisites.
2. Repeat Test 1.

**Expected:** DBeaver does not start after sign-in.

## Test 5: Less than one minute of uptime

1. Start DBeaver and restart Windows within 60 seconds.

**Expected:** DBeaver does not start after sign-in. This is Windows behavior, not a defect.

## Test 6: Crash does not relaunch

1. Start DBeaver and wait 60 seconds.
2. End the `dbeaver.exe` process in Task Manager.

**Expected:** Windows does not relaunch DBeaver.

## Test 7: Unsaved changes at shutdown

1. Open a SQL editor, type text, and do not save.
2. Wait 60 seconds and restart Windows.

**Expected:** Windows shows that DBeaver is preventing shutdown, because DBeaver asks about unsaved changes.
Click **Cancel** on the Windows screen, answer the DBeaver prompt, then restart again.
DBeaver reopens with the editor after sign-in.

## Test 8: Non-Windows platforms

1. Start DBeaver on macOS or Linux.

**Expected:** DBeaver starts and behaves as before. The error log has no restart registration messages.

## Troubleshooting

- Enable debug logging and look in the DBeaver log for `RegisterApplicationRestart failed` or `Error registering application restart`.
- Restored editors depend on **Window > Preferences > Editors > "Reopen database editors on restart"**, which is on by default.
