# Windows crashing issue

## Python installation

Reproduces on a fresh venv of:

`Python 3.7.0 (v3.7.0:1bf9cc5093, Jun 27 2018, 04:59:51) [MSC v.1914 64 bit (AMD64)] on win32`

(Did not crash on Linux or Mac when I tested it.)

## Repro steps

```
pip install otree
otree resetdb --noinput
```

## Expected result

Should output the following:

```
INFO Database engine: SQLite
INFO Retrieving Existing Tables...
INFO Dropping Tables...
INFO Creating Database 'default'...
Operations to perform:
  Synchronize unmigrated apps: auth, channels, contenttypes, demoapp, djhuey, forms, idmap, messages, otree, sessions, staticfiles
  Apply all migrations: (none)
Synchronizing apps without migrations:
  Creating tables...
    Creating table otree_pagecompletion
    Creating table otree_pagetimeout
    Creating table otree_completedgroupwaitpage
    Creating table otree_completedsubsessionwaitpage
    Creating table otree_participanttoplayerlookup
    Creating table otree_participantlockmodel
    Creating table otree_undefinedformmodel
    Creating table otree_roomtosession
    Creating table otree_failedsessioncreation
    Creating table otree_participantroomvisit
    Creating table otree_browserbotslaunchersessioncode
    Creating table otree_chatmessage
    Creating table otree_session
    Creating table otree_participant
    Creating table auth_permission
    Creating table auth_group
    Creating table auth_user
    Creating table django_content_type
    Creating table django_session
    Creating table demoapp_subsession
    Creating table demoapp_group
    Creating table demoapp_player
    Running deferred SQL...
Running migrations:
  No migrations to apply.
INFO Created new tables and columns.
```

## Actual result:

Python crashes, and either produces no output or just the first
2 lines of the above output.

Here is what I see in Windows Event Viewer:

```
Faulting application name: python.exe, version: 3.7.150.1013, time stamp: 0x5b331a30
Faulting module name: python37.dll, version: 3.7.150.1013, time stamp: 0x5b3319ec
Exception code: 0xc0000005
Fault offset: 0x000000000002a8fb
Faulting process id: 0x5788
Faulting application start time: 0x01d45e0b5ddc4025
Faulting application path: c:\otree\ve-debug\scripts\python.exe
Faulting module path: c:\otree\ve-debug\scripts\python37.dll
Report Id: a5dded9f-409f-4118-9d8b-fe41a36119cb
Faulting package full name:
Faulting package-relative application ID:
```

```
Fault bucket 1565749660316999997, type 4
Event Name: APPCRASH
Response: Not available
Cab Id: 0

Problem signature:
P1: python.exe
P2: 3.7.150.1013
P3: 5b331a30
P4: python37.dll
P5: 3.7.150.1013
P6: 5b3319ec
P7: c0000005
P8: 000000000002a8fb
P9:
P10:

Attached files:
\\?\C:\ProgramData\Microsoft\Windows\WER\Temp\WER3B76.tmp.dmp
\\?\C:\ProgramData\Microsoft\Windows\WER\Temp\WER3C13.tmp.WERInternalMetadata.xml
\\?\C:\ProgramData\Microsoft\Windows\WER\Temp\WER3C53.tmp.xml
\\?\C:\ProgramData\Microsoft\Windows\WER\Temp\WER3C6D.tmp.csv
\\?\C:\ProgramData\Microsoft\Windows\WER\Temp\WER3CBC.tmp.txt

These files may be available here:
C:\ProgramData\Microsoft\Windows\WER\ReportArchive\AppCrash_python.exe_71b4ff563e6039a8af8fc37cbea29d23f7bf5_7cdba627_766c4450

Analysis symbol:
Rechecking for solution: 0
Report Id: a5dded9f-409f-4118-9d8b-fe41a36119cb
Report Status: 268435456
Hashed bucket: fdbfa3e49012c12245baa9053662213d
Cab Guid: 0
```

## Troubleshooting

I found that I can prevent the crash by slightly changing a string in `demoapp/templates/demoapp/MyPage.html`.

That file contains:

`{% formfield player.a label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}`

It seems the `label` is causing the crash, although I tried changing the string in many ways
and cannot spot the pattern.

### Does not crash

Replace the above with any of the following lines and it doesn't crash:

```
{% formfield player.want_to_provide_payment_details label="¿DesdeaAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.want_to_provide_payment_details label="¿AAAAAAAAAAAA A" %}
{% formfield player.want_to_provide_payment_details label="¿AAAAAAAAAAAA AAAAAAAAAAAAA" %}
{% formfield player.want_to_provide_payment_details label="¿AAAAAAAAAAAA AAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.want_to_provide_payment_details label="AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.want_to_provide_payment_details label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.want_to_provide_payment_details label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.want_to_provide_payment_details label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.a label="AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA¿" %}
{% formfield player.minimum_tolerated_amount label="Mínimo de puntos que está dispuesto a recibir del participante A" %}
{% formfield player.minimum_tolerated_amount label="Mínimo AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.minimum_tolerated_amount label="íAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.minimum_tolerated_amount label="íAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
```

### Crashes

Any of the following lines *do* trigger the crash.

```
{% formfield player.a label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.a label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %} (wrote myself)
{% formfield player.a label="AAAAAAAA¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.a label="AAAAAAAAAAAAAAAAAAAAAA¿AAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.a label="A¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.want_to_provide_payment_details label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.want_to_provide_payment_details label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.want_to_provide_payment_details label="¿AAAAAAAAAAAA AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.want_to_provide_payment_details label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.a label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.a label="ÁAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.a label="AíAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
```

### Sometimes crashes

```
{% formfield player.a label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.a label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.a label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
{% formfield player.a label="¿AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" %}
```

### Tracing results

Using the ``trace`` module I found that Python usually stops in
the function `otree.checks.template_content_is_in_blocks()`, but I cannot see why.