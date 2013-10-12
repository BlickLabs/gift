# Gift [![Build Status](https://secure.travis-ci.org/notatestuser/gift.png?branch=master)](http://travis-ci.org/notatestuser/gift)

A simple Node.js wrapper for the Git CLI. The API is based on
[Grit](https://github.com/mojombo/grit)

# Installation (of this fresher fork)

    $ npm install gift@>0.0.6

# API

For existing repositories:

    git  = require 'gift'

    repo = git "path/to/repo"
    # => #<Repo>

Initialize a new repository:

    git = require 'gift'

    git.init "path/to/repo", (err, _repo) ->
      repo = _repo
      # => #<Repo>

Initialize a new bare repository:

    git = require 'gift'

    git.init "path/to/bare/repo", true, (err, _repo) ->
      repo = _repo
      # => #<Repo>

Clone a repository:

    git = require 'gift'

    git.clone "git@host:path/to/remote/repo.git", "path/to/local/clone/repo", (err, _repo) ->
      repo = _repo
      # => #<Repo>

## Repo
### `Repo#path`
`String` - The path to the repository.

### `Repo#commits([treeish, [limit, [skip, ]]]callback)`
Get a list of commits.

  * `treeish`  - `String`  (optional).
  * `limit`    - `Integer` (optional).
  * `skip`     - `Integer` (optional).
  * `callback` - `Function` which receives `(err, commits)`, where `commits` is
                 an `Array` of `Commit`s.

Get the 10 most recent commits to master.

    repo.commits (err, commits) ->

Or to a different tag or branch.

    repo.commits "v0.0.3", (err, commits) ->

Limit the maximum number of commits returned.

    repo.commits "master", 30, (err, commits) ->

Skip some (for pagination):

    repo.commits "master", 30, 30, (err, commits) ->

### `Repo#tree([treeish]) => Tree`
The `Tree` object for the treeish (which defaults to "master").

    repo.tree().contents (err, children) ->
      for child in children
        console.log child.name

### `Repo#diff(commitA, commitB, [paths, ]callback)`
Get the difference between the trees.

The callback receives `(err, diffs)`.

### `Repo#identity(callback)`
Get the commit identity for this repository.

The callback receives `(err, actor)`, where `actor` is an Actor.

### `Repo#identify(actor, callback)`
Set your account's default identity for commits to this repository.

The callback receives `(err)`.

### `Repo#remotes(callback)`
Get the repository's remotes.

Receives `(err, remotes)`, where each remote is a Ref.

### `Repo#remote_list(callback)`
Get a list of the repository's remote names.

Get the string names of each of the remotes.

### `Repo#remote_add(name, url, callback)`
Equivalent to `git remote add <name> <url>`.

### `Repo#remote_remove(name, callback)`
Remove a remote.

### `Repo#remote_fetch(name, callback)`
`git fetch <name>`

### `Repo#remote_push(name, callback)`
`git push <name>`

### `Repo#status(callback)`
Uses `--porcelain` to parse repository status in a way that is agnostic of system language. The callback receives `(err, status)`. See below for a definition of what `status` is.

### `Repo#config(callback)`
`git config` parsed as a simple, one-level object. The callback receives `(err, config)`.

### `Repo#create_branch(name, callback)`
Create a new branch with `name`, and call the callback when complete
with an error, if one occurred.

### `Repo#delete_branch(name, callback)`
Delete the branch `name`, and call the callback with an error, if one occurred.

### `Repo#tags(callback)`
Get a list of `Tag`s.

### `Repo#create_tag(name, [options, ]callback)`
Create a tab with the given name.

### `Repo#delete_tag(name, callback)`
Delete the tag with the given name.

### `Repo#branches(callback)`
`callback` receives `(err, heads)`.

### `Repo#create_branch(name, callback)`
Create a branch with the given name.

### `Repo#delete_branch(delete, callback)`
Delete the branch with the given name.

### `Repo#branch([branch, ]callback)`
Get a branch.

  * `branch`   - The name of the branch to get. Defaults to the
                 currently checked out branch.
  * `callback` - Receives `(err, head)`.


### `Repo#commit(message, [options, ]callback)`
Commit some changes.

  * `message`  - `String`
  * `options`  -
    - `all`    - `Boolean`
    - `amend`  - `Boolean`
  * `callback` - Receives `(err)`.

### `Repo#add(files, callback)`
`git add <files>`

### `Repo#remove(files, callback)`
`git rm <files>`

### `Repo#checkout(treeish, callback)`
`git checkout <treeish>`

### `Repo#sync([[remote, ]branch, ]callback)`
Sync the current branch with the remote, keeping all local changes intact.

The following steps are carried out: `stash`, `pull`, `push`, `stash pop`. If there were no changes to stash, the last `stash pop` is not executed.

  * `remote`   - `String` (defaults to `origin`).
  * `branch`   - `String` (defaults to `master`).
  * `callback` - Receives `(err)`.


## Commit
### `Commit#id`
`String` - The commit's SHA.

### `Commit#parents`
`Commit[]`

### `Commit#tree(callback)`

  * `callback` - Receives `(err, tree)`.

### `Commit#author`
`Actor`
### `Commit#authored_date`
`Date`
### `Commit#committer`
`Actor`
### `Commit#committed_date`
`Date`
### `Commit#message`
`String`


## Head
### `Head#name`
`String`

### `Head#commit`
`Commit`

## Tag
### `Tag#name`
`String`

### `Tag#commit`
`Commit`

### `Tag#message(callback)`
The callback receives `(err, message)` (`message` is a String).

### `Tag#tagger(callback)`
The callback receives `(err, actor)`.

### `Tag#tag_date(callback)`
The callback receives `(err, date)`.

## Config
### `Config#items`
`Object` - The keys are dotted precisely as the console output from `git config`. E.g., `{'user.name': 'John Doe'}`

## Status
### `Status#clean`
`Boolean`

### `Status#files`
`Object` - The keys are files, the values objects indicating whether or not
the file is staged, tracked, etc.

Each file has the following properties:

  * `type`    - "A" for added, "M" for modified, "D" for deleted.
  * `staged`  - `Boolean`
  * `tracked` - `Boolean`

## Actor
### `Actor#name`
`String`

### `Actor#email`
`String`

### `Actor#hash`
`String` - The MD5 hash of the actor's email. Useful for displaying
[Gravatar](http://en.gravatar.com/) avatars.


## Tree
### `Tree#id`
`String` - SHA1

### `Tree#contents(callback)`

  * `callback` - Receives `(err, children)`.
  * `children` - An array of `Blob`s, `Tree`s, and `Submodule`s.

### `Tree#blobs(callback)`

  * `callback` - Receives `(err, child_blobs)`.
  * `children` - `[Blob]`

### `Tree#trees(callback)`

  * `callback` - Receives `(err, child_trees)`.
  * `children` - `[Tree]`

### `Tree#find(directory, callback)`

  * `directory` - `String`
  * `callback`  - Receives `(err, thing)`.

## Blob
### `Blob#id`
`String` - SHA1

### `Blob#mode`
`String`

### `Blob#data(callback)`

  * `callback` - `(err, data)`

## Submodule
### `Submodule#id`
`String`

### `Submodule#name`
`String`

### `Submodule#mode`
`String`

### `Submodule#url(callback)`
Get the url the submodule points to.


# License
See LICENSE.
