# Administration

## Making commits show up on issues they refer to

When this works, the issue will have a separated tab called `Associated revisions` with a list of commits. But making this work is tricky

1. `git clone --bare` a git project to a place local and accessible to the redmine server *(and you will need to make some service that updates it, it won't be updated automatically)*. Yeah, the `--bare` option is required.
2. In a Redmine project go to a tab called `Repository`. Create a new one, choose `Git` and fill out various forms, should be straightforward. After being created, check that it works by visiting the project and seeing files, revisions, etc…
3. Change **global** settings, by going to Administration → Settings → Repositories, and using inside `Referencing keywords` as a keyword just an asterisk `*`. It's a hack that will allow use to use sane syntax to reference tasks in your commits.

Now, to refer an issue `777` use a `#777` text somewhere inside your commit message. The issue should be linked automatically. Note that sometimes it takes time for redmine to update information, but you can force it by going inside your Redmine project to the repository, and then opening the commit you're interested in. If everything works correctly, under a title `Related issues` you will a link to the issue, and then reloading the page with the issue will make the link appear.
