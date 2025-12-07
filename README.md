# coralogix-backups

A tool for taking backups of Coralogix instances, versioning them, and restoring from those backups.

Run most-simply, it will download all your dashboards, alerts, e2m definitions and parsing rules to local files.

A `cx-backups.d` directory will be created and inside there one more for each thing backed up - `alerts`, `dashboards`,
`events2metrics` and `parsing-rules`.

Each of these will contain one JSON file per thing-backed-up; one for each alert, dashboard, e2m rule and parsing-rule
group.

Additionally, a `dashboards.yaml` will be written, which attempts to describe the dashboards, and the `dashboard-catalog.json`
and `alert-definitions.json` as returned from the API.

It needs a `CX_API_KEY` and `CX_REGION` environment variable set, but most-simply if you run

    CX_API_KEY=cxup_AbcDeFGHIjklMNopQRSTuvWXyZ CX_REGION=EU2 cx-backups

you will find a `cx-backups.d` directory is created in your current directory, with a series of JSON files representing
the setup of your team. You can set the `CX_BACKUPS_WORKDIR` env var to put them elsewhere.

# Restoring

Restoring is done one file at a time, the usage is:

    cx-backups restore [path-to-file]

where the file is a file defining a single backed-up entity; one dashboard, one alert or one e2m rule. Parsing Rules are not
supported yet, and nor are dashboard folders.

# Git version-tracking

If the `CX_BACKUPS_GIT_REPO_URL` environment variable is set (to the URL to a git repo you can push to), cx-backups will clone
that repo before backing up, then take the backup into that directory, before creating a commit and pushing that back up to the
origin.

This is is intended to be run as a periodic job so as to be able to track changes - see when that value was set, or what that
was set to six weeks ago or something.

The commit message defaults to 'cx-backups' appended with the date, but you can set it to something else with the `CX_BACKUPS_GIT_COMMIT_MESSAGE`
environment variable.

By default, the files for things that are have been removed since the last backup are `git rm`ed; you can keep them by setting
the `CX_BACKUPS_NO_DELETE_REMOVED` env var to anything.

# Installation and running

On something Debian based you need `sudo apt-get install python3 python3-requests python3-yaml python3-json`

On anything else, get Python installed and do `pip install -r requirements.txt`

There are no supported argument options; everything is set via environment variables:

Required:
  * CX_API_KEY - your Coralogix API key for API access. Should be a Personal key, not a send-your-data one.
  * CX_REGION  - the three-character name of the region your CX is in (EU0, EU2, etc.)

Optional:
  * CX_BACKUPS_DO_GIT             - Use git locally; set CX_BACKUPS_GIT_REPO_URL to use a remote repository, set this to just use git locally in the backups dir
  * CX_BACKUPS_WORKDIR            - Path to the dir to write backups to. Defaults to ./cx-backups.d unless git is being used where the default is instead /tmp/cx-backups
  * CX_BACKUPS_GIT_REPO_URL       - SSH URL of the git repo to commit to. If neither this nor CX_BACKUPS_DO_GIT is set, git is not used at all
  * CX_BACKUPS_GIT_COMMIT_MESSAGE - The commit message to use on git operations; defaults to 'cx-backups ' appended with the date
  * CX_BACKUPS_NO_DELETE_REMOVED  - By default, json backup files for things that have been deleted are git-rmed; this supresses that behaviour

# Limitations

There are a few. This is partly written because I needed to do something, and the rest is because given that first thing, how hard can another be?

* Cannot restore events2metrics
* Cannot restore dashboard folders
* Cannot restore parsing rules

If you really want these, shout and I'll probably get around to them.
