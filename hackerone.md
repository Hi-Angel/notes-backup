# RCE techniques

1. Rewriting dex file an app might be loading *(whereas dex file holds the code to execute)*.

# possible places for RCE

1. Debugging logs that allows to modify the target folder and content.

# Mail.ru calendar

Idea: different mail.ru apps probably should share some common code. I have to decompile another one, and compare the file list.

1. there is a `ru/mail/util/log/` dir.

-------

1. `FileSendingProgressDialog` can I send a file to other device forcing RCE?

## Misc

*Look at what `ru/mail/util/log/logger/SendLogsService.java` does.* — It simply sends files somewhere. Unless I can do something special by changing the address, but then I also need to change content of files for it to be useful.

`FileLogger` is worthless, unless I can exploit android random number generator to insert a certain path instead of `%u` in `FileHandler`.

`deleteFiles(File paramFile)` & co at `splashScreenActivity.java` — do I care?

`org/codehaus/jackson/JsonFactory.java` — probably writes json files. Worth taking a look.

`org/codehaus/jackson/map/ObjectWriter.java:461:  public void writeValue(File paramFile, Object paramObject)`

`org/apache/http/entity/FileEntity.java:18:  public FileEntity(File paramFile)
`
I must probably look `ru/` subdir because `org/` one is external libs, and it is the usage that is vulnerable.
