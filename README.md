# git-local-backup
A solution to automatically backup git repos on local / cloud-synced folders (Nextcloud, Dropbox, Google Cloud, etc.)

## How to enable backup in a git repository
1. Clone `git-local-backup` repository:
    ```
    $ git clone https://github.com/dcommisso/git-local-backup.git
    ```

2. Link (or copy) `hooks/post-commit` to the repo to backup:
    ```
    $ ln -s git-local-backup/hooks/post-commit <YOUR-REPO>/.git/hooks/post-commit
    ```

3. Enter your repo and configure the directory where the backups will be saved:
    ```
    $ cd <YOUR-REPO>

    $ git config set gitlocalbackup.directory <BACKUP-DIRECTORY>
    ```

4. (Optional) Set the number of bundle files to keep:
    ```
    $ git config set gitlocalbackup.backupstokeep <NUMBER-OF-BACKUPS>
    ```
   Since a git bundle already contains all the repo's history, there's really no need to keep a lot of bundles, a value of 1 or 2 should be fine.

### Configuring backup for new repositories
It's also possible to use git-local-backup repo as template for new repository, in order to create and install backup using just one command:

    ```
    $ git init --template=git-local-backup <NEW-REPOSITORY>
    ```

## Restore backup
Just run `git clone` on bundle file:

```
$ git clone <BACKUP-DIR>/<REPO-NAME>.bundle
```

## Uninstall git-local-backup
Just delete the `post-commit` script in your repository:

```
$ rm <YOUR-REPO>/.git/hooks/post-commit
```

## Rationale
I find it very convenient to use git for single user personal projects (not only programming but also for other stuff, e.g. literature projects or miscellaneous documentation). I don't need synchronization with other users but I still need a backup. Since it's private stuff I don't want to use public git servers, but at the same time I also don't want the hassle of maintaining (and backupping) a private git server or to encrypt them. The ideals solution for me is using local directories synchronized with private cloud, but syncing a git repo in this way would most likely break it.
So I ended up writing a bash script  configured as `post-commit` git hook that runs `git bundle` after each commit, plus some logic to avoid cluttering backup folder with a lot of useless backups.

## How it works
Since the script is configured as `post-commit` git hook, it will be automatically executed after each commit. The script runs `git bundle create --all` and saves the bundle in backup directory defined in `gitlocalbackup.directory`, creates/moves the link to the last backup file and deletes older bundles (if configured) according to `gitlocalbackup.backupstokeep` git option. Please check the console logs after each commit to make sure the backup worked properly. 

## Sync a repository between devices
If the directory where the backup is stored is accessible by other devices it's possible to keep repositories in sync:

```
$ git clone <BACKUP-DIR>/<REPO-NAME>.bundle

... the repository is updated and backupped from a different device

$ git pull
```

however this is dangerous and shouldn't be abused: if the shared directory is not promptly updated there is a risk of losing data. If a robust sync is needed, it's better to rely on a git server.
