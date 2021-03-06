# octonode

octonode is a library for nodejs to access the [github v3 api](http://developer.github.com)

## Installation
```
npm install octonode
```

## Usage

```js
var github = require('octonode');

// Then we instanciate a client with or without a token (as show in a later section)

var ghme   = client.me();
var ghuser = client.user('pksunkara');
var ghrepo = client.repo('pksunkara/hub');
var ghorg  = client.org('flatiron');
var ghissue = client.issue('pksunkara/hub', 37);
var ghpr = client.pr('pksunkara/hub', 37);
var ghgist = client.gist();
var ghteam = client.team(37);

var ghsearch = client.search();
```

#### Build a client which accesses any public information

```js
var client = github.client();

client.get('/users/pksunkara', {}, function (err, status, body, headers) {
  console.log(body); //json object
});
```

#### Build a client from an access token

```js
var client = github.client('someaccesstoken');

client.get('/user', {}, function (err, status, body, headers) {
  console.log(body); //json object
});
```

#### Build a client from credentials

```js
var client = github.client({
  username: 'pksunkara',
  password: 'password'
});

client.get('/user', {}, function (err, status, body, headers) {
  console.log(body); //json object
});
```

#### Build a client from client keys

```js
var client = github.client({
  id: 'abcdefghijklmno',
  secret: 'abcdefghijk'
});

client.get('/user', {}, function (err, status, body, headers) {
  console.log(body); //json object
});
```


## Request Options

Request options can be set by setting defaults on the client. (e.g. Proxies)

```js
var client = github.client();

client.requestDefaults['proxy'] = 'https://myproxy.com:1085'
```
These options are passed though to `request`, see their API here: https://github.com/mikeal/request#requestoptions-callback

### Proxies
You can set proxies dynamically by using the example above, but Octonode will respect environment proxies by default. Just set this using:
`export HTTP_PROXY='https://myproxy.com:1085'` if you are using the command line

__Many of the below use cases use parts of the above code__

## Authentication

#### Authenticate to github in cli mode (desktop application)

```js
github.auth.config({
  username: 'pksunkara',
  password: 'password'
}).login(['user', 'repo', 'gist'], function (err, id, token) {
  console.log(id, token);
});
```

#### Revoke authentication to github in cli mode (desktop application)

```js
github.auth.config({
  username: 'pksunkara',
  password: 'password'
}).revoke(id, function (err) {
  if (err) throw err;
});
```

#### Authenticate to github in web mode (web application)

```js
// Web application which authenticates to github
var http = require('http')
  , url = require('url')
  , qs = require('querystring')
  , github = require('octonode');

// Build the authorization config and url
var auth_url = github.auth.config({
  id: 'mygithubclientid',
  secret: 'mygithubclientsecret'
}).login(['user', 'repo', 'gist']);

// Store info to verify against CSRF
var state = auth_url.match(/&state=([0-9a-z]{32})/i);

// Web server
http.createServer(function (req, res) {
  uri = url.parse(req.url);
  // Redirect to github login
  if (uri.pathname=='/login') {
    res.writeHead(301, {'Content-Type': 'text/plain', 'Location': auth_url})
    res.end('Redirecting to ' + auth_url);
  }
  // Callback url from github login
  else if (uri.pathname=='/auth') {
    var values = qs.parse(uri.query);
    // Check against CSRF attacks
    if (!state || state[1] != values.state) {
      res.writeHead(403, {'Content-Type': 'text/plain'});
      res.end('');
    } else {
      github.auth.login(values.code, function (err, token) {
        res.writeHead(200, {'Content-Type': 'text/plain'});
        res.end(token);
      });
    }
  } else {
    res.writeHead(200, {'Content-Type': 'text/plain'})
    res.end('');
  }
}).listen(3000);

console.log('Server started on 3000');
```

## Rate Limiting

You can also check your rate limit status by calling the following.

```js
client.limit(function (err, left, max) {
  console.log(left); // 4999
  console.log(max);  // 5000
});
```

## API Callback Structure

__All the callbacks for the following will take first an error argument, then a data argument, like this:__

```js
ghme.info(function(err, data, headers) {
  console.log("error: " + err);
  console.log("data: " + data);
  console.log("headers:" + headers);
});
```

## Pagination

If a function is said to be supporting pagination, then that function can be used in many ways as shown below. Results from the function are arranged in [pages](http://developer.github.com/v3/#pagination).

The page argument is optional and is used to specify which page of issues to retrieve.
The perPage argument is also optional and is used to specify how many issues per page.

```js
// Normal usage of function
ghrepo.issues(callback); //array of first 30 issues

// Using pagination parameters
ghrepo.issues(2, 100, callback); //array of second 100 issues
ghrepo.issues(10, callback); //array of 30 issues from page 10

// Pagination parameters can be set with query object too
ghrepo.issues({
  page: 2,
  per_page: 100,
  state: 'closed'
}, callback); //array of second 100 issues which are closed
```

## Github authenticated user api

Token/Credentials required for the following:

#### Get information about the user (GET /user)

```js
ghme.info(callback); //json
```

#### Update user profile (PATCH /user)

```js
ghme.update({
  "name": "monalisa octocat",
  "email": "octocat@github.com",
}, callback);
```

#### Get emails of the user (GET /user/emails)

```js
ghme.emails(callback); //array of emails
```

#### Set emails of the user (POST /user/emails)

```js
ghme.emails(['new1@ma.il', 'new2@ma.il'], callback); //array of emails
ghme.emails('new@ma.il', callback); //array of emails
```

#### Delete emails of the user (DELETE /user/emails)

```js
ghme.emails(['new1@ma.il', 'new2@ma.il']);
ghme.emails('new@ma.il');
```

#### Get the followers of the user (GET /user/followers)

```js
ghme.followers(callback); //array of github users
```

#### Get users whom the user is following (GET /user/following)

This query supports [pagination](#pagination).

```js
ghme.following(callback); //array of github users
```

#### Check if the user is following a user (GET /user/following/marak)

```js
ghme.following('marak', callback); //boolean
```

#### Follow a user (PUT /user/following/marak)

```js
ghme.follow('marak');
```

#### Unfollow a user (DELETE /user/following/marak)

```js
ghme.unfollow('marak');
```

#### Get public keys of a user (GET /user/keys)

```js
ghme.keys(callback); //array of keys
```

#### Get a single public key (GET /user/keys/1)

```js
ghme.keys(1, callback); //key
```

#### Create a public key (POST /user/keys)

```js
ghme.keys({"title":"laptop", "key":"ssh-rsa AAA..."}, callback); //key
```

#### Update a public key (PATCH /user/keys/1)

```js
ghme.keys(1, {"title":"desktop", "key":"ssh-rsa AAA..."}, callback); //key
```

#### Delete a public key (DELETE /user/keys/1)

```js
ghme.keys(1);
```

#### Get the starred repos for the user (GET /user/starred)

This query supports [pagination](#pagination).

```js
ghme.starred(callback); //array of repos
```

#### Check if you have starred a repository (GET /user/starred/pksunkara/octonode)

```js
ghme.starred('flatiron/flatiron', callback); //boolean
```

#### Star a repository (PUT /user/starred/pksunkara/octonode)

```js
ghme.star('flatiron/flatiron');
```

#### Unstar a repository (DELETE /user/starred/pksunkara/octonode)

```js
ghme.unstar('flatiron/flatiron');
```

#### Get the subscriptions of the user (GET /user/subscriptions)

This query supports [pagination](#pagination).

```js
ghme.watched(callback); //array of repos
```

#### List your public and private organizations (GET /user/orgs)

This query supports [pagination](#pagination).

```js
ghme.orgs(callback); //array of orgs
```

#### List your repositories (GET /user/repos)

This query supports [pagination](#pagination).

```js
ghme.repos(callback); //array of repos
```

#### Create a repository (POST /user/repos)

```js
ghme.repo({
  "name": "Hello-World",
  "description": "This is your first repo",
}, callback); //repo
```

#### Fork a repository (POST /repos/pksunkara/hub/forks)

```js
ghme.fork('pksunkara/hub', callback); //forked repo
```

## Github users api

No token required for the following

#### Get information about a user (GET /users/pksunkara)

```js
ghuser.info(callback); //json
```

#### Get user followers (GET /users/pksunkara/followers)

This query supports [pagination](#pagination).

```js
ghuser.followers(callback); //array of github users
```

#### Get user followings (GET /users/pksunkara/following)

This query supports [pagination](#pagination).

```js
ghuser.following(callback); //array of github users
```

#### Get events performed by a user (GET /users/pksunkara/events)

This query supports [pagination](#pagination).

```js
ghuser.events(['commit_comment'], callback); //array of events
```

#### Get user public organizations (GET /users/pksunkara/orgs)

This query supports [pagination](#pagination).

```js
ghuser.orgs(callback); //array of organizations
```

## Github repositories api

#### Get information about a repository (GET /repos/pksunkara/hub)

```js
ghrepo.info(callback); //json
```

#### Get the collaborators for a repository (GET /repos/pksunkara/hub/collaborators)

```js
ghrepo.collaborators(callback); //array of github users
```

#### Check if a user is collaborator for a repository (GET /repos/pksunkara/hub/collaborators/marak)

```js
ghrepo.collaborators('marak', callback); //boolean
```

#### Get the commits for a repository (GET /repos/pksunkara/hub/commits)

```js
ghrepo.commits(callback); //array of commits
```

#### Get a certain commit for a repository (GET /repos/pksunkara/hub/commits/18293abcd72)
```js
ghrepo.commit('18293abcd72', callback); //commit
```

#### Get the tags for a repository (GET /repos/pksunkara/hub/tags)

```js
ghrepo.tags(callback); //array of tags
```

#### Get the releases for a repository (GET /repos/pksunkara/hub/releases)

```js
ghrepo.releases(callback); //array of releases
```

#### Get the languages for a repository (GET /repos/pksunkara/hub/languages)

```js
ghrepo.languages(callback); //array of languages
```

#### Get the contributors for a repository (GET /repos/pksunkara/hub/contributors)

```js
ghrepo.contributors(callback); //array of github users
```

#### Get the branches for a repository (GET /repos/pksunkara/hub/branches)

```js
ghrepo.branches(callback); //array of branches
```

#### Get the issues for a repository (GET /repos/pksunkara/hub/issues)

This query supports [pagination](#pagination).

```js
ghrepo.issues(callback); //array of issues
```

#### Create an issue for a repository (POST /repos/pksunkara/hub/issues)

```js
ghrepo.issue({
  "title": "Found a bug",
  "body": "I'm having a problem with this.",
  "assignee": "octocat",
  "milestone": 1,
  "labels": ["Label1", "Label2"]
}, callback); //issue
```

#### Get the pull requests for a repository (GET /repos/pksunkara/hub/pulls)

This query supports [pagination](#pagination).

```js
ghrepo.prs(callback); //array of pull requests
```

#### Create a pull request (POST /repos/pksunkara/hub/pulls)

```js
ghrepo.pr({
  "title": "Amazing new feature",
  "body": "Please pull this in!",
  "head": "octocat:new-feature",
  "base": "master"
}, callback); //pull request
```

#### Get the hooks for a repository (GET /repos/pksunkara/hub/hooks)

This query supports [pagination](#pagination).

```js
ghrepo.hooks(callback); //array of hooks
```

#### Create a hook (POST /repos/pksunkara/hub/hooks)

```js
ghrepo.hook({
  "name": "web",
  "active": true,
  "events": ["push", "pull_request"],
  "config": {
    "url": "http://myawesomesite.com/github/events"
  }
}, callback); // hook
```

#### Get the README for a repository (GET /repos/pksunkara/hub/readme)

```js
ghrepo.readme(callback); //file
ghrepo.readme('v0.1.0', callback); //file
```

#### Get the contents of a path in repository

```js
ghrepo.contents('lib/index.js', callback); //path
ghrepo.contents('lib/index.js', 'v0.1.0', callback); //path
```

#### Create a file at a path in repository

```js
ghrepo.createContents('lib/index.js', 'commit message', 'content', callback); //path
ghrepo.createContents('lib/index.js', 'commit message', 'content', 'v0.1.0', callback); //path
```

#### Update a file at a path in repository

```js
ghrepo.updateContents('lib/index.js', 'commit message', 'content', 'put-sha-here', callback); //path
ghrepo.updateContents('lib/index.js', 'commit message', 'content', 'put-sha-here', 'v0.1.0', callback); //path
```

#### Delete a file at a path in repository

```js
ghrepo.deleteContents('lib/index.js', 'commit message', 'put-sha-here', callback); //path
ghrepo.deleteContents('lib/index.js', 'commit message', 'put-sha-here', 'v0.1.0', callback); //path
```

#### Get archive link for a repository

```js
ghrepo.archive('tarball', callback); //link to archive
ghrepo.archive('zipball', 'v0.1.0', callback); //link to archive
```

#### Get the blob for a repository (GET /repos/pksunkara/hub/git/blobs/SHA)

```js
ghrepo.blob('18293abcd72', callback); //blob
```

#### Get users who starred a repository (GET /repos/pksunkara/hub/stargazers)

```js
ghrepo.stargazers(1, 100, callback); //array of users
ghrepo.stargazers(10, callback);     //array of users
ghrepo.stargazers(callback);         //array of users
```

#### Get the teams for a repository (GET /repos/pksunkara/hub/teams)

```js
ghrepo.teams(callback); //array of teams
```

#### Get a git tree (GET /repos/pksunkara/hub/git/trees/18293abcd72)

```js
ghrepo.tree('18293abcd72', callback); //tree
ghrepo.tree('18293abcd72', true, callback); //recursive tree
```

#### Delete the repository (DELETE /repos/pksunkara/hub)

```js
ghrepo.destroy();
```

#### List statuses for a specific ref (GET /repos/pksunkara/hub/statuses/master)

```js
ghrepo.statuses('master', callback); //array of statuses
```

#### Create status (POST /repos/pksunkara/hub/statuses/SHA)

```js
ghrepo.status('18e129c213848c7f239b93fe5c67971a64f183ff', {
  "state": "success",
  "target_url": "http://ci.mycompany.com/job/hub/3",
  "description": "Build success."
}, callback); // created status
```

## Github organizations api

#### Get information about an organization (GET /orgs/flatiron)

```js
ghorg.info(callback); //json
```

#### Update an organization (POST /orgs/flatiron)

```js
ghorg.update({
  blog: 'https://blog.com'
}, callback); // org
```

#### List organization repositories (GET /orgs/flatiron/repos)

This query supports [pagination](#pagination).

```js
ghorg.repos(callback); //array of repos
```

#### Create an organization repository (POST /orgs/flatiron/repos)

```js
ghorg.repo({
  name: 'Hello-world',
  description: 'My first world program'
}, callback); //repo
```

#### Get an organization's teams (GET /orgs/flatiron/teams)

```js
ghorg.teams(callback); //array of teams
```

#### Get an organization's members (GET /orgs/flatiron/members)

```js
ghorg.members(callback); //array of github users
```

#### Check an organization member (GET /orgs/flatiron/members/pksunkara)

```js
ghorg.member('pksunkara', callback); //boolean
```

## Github issues api

#### Get a single issue (GET /repos/pksunkara/hub/issues/37)

```js
ghissue.info(callback); //issue
```

#### Edit an issue for a repository (PATCH /repos/pksunkara/hub/issues/37)

```js
ghissue.update({
  "title": "Found a bug and I am serious",
}, callback); //issue
```

#### List comments on an issue (GET /repos/pksunkara/hub/issues/37/comments)

This query supports [pagination](#pagination).

```js
ghissue.comments(callback); //array of comments
```

## Github pull requests api

#### Get a single pull request (GET /repos/pksunkara/hub/pulls/37)

```js
ghpr.info(callback); //pull request
```

#### Update a pull request (PATCH /repos/pksunkara/hub/pulls/37)

```js
ghpr.update({
  'title': 'Wow this pr'
}, callback); //pull request
```

#### Close a pull request

```js
ghpr.close(callback); //pull request
```

#### Get if a pull request has been merged (GET /repos/pksunkara/hub/pulls/37/merge)

```js
ghpr.merged(callback); //boolean
```

#### List commits on a pull request (GET /repos/pksunkara/hub/pulls/37/commits)

```js
ghpr.commits(callback); //array of commits
```

#### List comments on a pull request (GET /repos/pksunkara/hub/pulls/37/comments)

```js
ghpr.comments(callback); //array of comments
```

#### List files in pull request (GET /repos/pksunkara/hub/pulls/37/files)

```js
ghpr.files(callback); //array of files
```

## Github gists api

#### List authenticated user's gists (GET /gists)

This query supports [pagination](#pagination).

```js
ghgist.list(callback); //array of gists
```

#### List authenticated user's public gists (GET /gists/public)

This query supports [pagination](#pagination).

```js
ghgist.public(callback); //array of gists
```

#### List authenticated user's starred gists (GET /gists/starred)

This query supports [pagination](#pagination).

```js
ghgist.starred(callback); //array of gists
```

#### List a user's public gists (GET /users/pksunkara/gists)

This query supports [pagination](#pagination).

```js
ghgist.user('pksunkara', callback); //array of gists
```

#### Get a single gist (GET /gists/37)

```js
ghgist.get(37, callback); //gist
```

#### Create a gist (POST /gists)

```js
ghgist.create({
  description: "the description",
  files: { ... }
}), callback); //gist
```

#### Edit a gist (PATCH /gists/37)

```js
ghgist.edit(37, {
  description: "hello gist"
}, callback); //gist
```

#### Delete a gist (DELETE /gists/37)

```js
ghgist.delete(37);
```

#### Fork a gist (POST /gists/37/forks)

```js
ghgist.fork(37, callback); //gist
```

#### Star a gist (PUT /gists/37/star)

```js
ghgist.star(37);
```

#### Unstar a gist (DELETE /gists/37/unstar)

```js
ghgist.unstar(37);
```

#### Check if a gist is starred (GET /gists/37/star)

```js
ghgist.check(37); //boolean
```

#### List comments on a gist (GET /gists/37/comments)

```js
ghgist.comments(37, callback); //array of comments
```

#### Create a comment (POST /gists/37/comments)

```js
ghgist.comments(37, {
  body: "Just commenting"
}, callback); //comment
```

#### Get a single comment (GET /gists/comments/1)

```js
ghgist.comment(1, callback); //comment
```

#### Edit a comment (POST /gists/comments/1)

```js
ghgist.comment(1, {
  body: "lol at commenting"
}, callback); //comment
```

#### Delete a comment (DELETE /gists/comments/1)

```js
ghgist.comment(1);
```

## Github teams api

#### Get a team (GET /team/37)

```js
ghteam.info(callback); //json
```

#### Get the team members (GET /team/37/members)

```js
ghteam.members(callback); //array of github users
```

#### Check if a user is part of the team (GET /team/37/members/pksunkara)

```js
ghteam.member('pksunkara'); //boolean
```

## Github search api

#### Search issues

```js
ghsearch.issues('pksunkara/hub', 'open', 'git', callback); //array of issues
```

#### Search repositories

```js
ghsearch.repos('git', 'javascript', 1, callback); //array of repositories
```

#### Search users

```js
ghsearch.users('git', callback); //array of users
```

#### Search emails

```js
ghsearch.emails('pavan.sss1991@gmail.com', callback); //user
```

## Testing
```
npm test
```

If you like this project, please watch this and follow me.

## Contributors
Here is a list of [Contributors](http://github.com/pksunkara/octonode/contributors)

### TODO

The following method names use underscore as an example. The library contains camel cased method names.

```js

// public repos for unauthenticated, private and public for authenticated
me.get_watched_repositories(callback);
me.is_watching('repo', callback);
me.start_watching('repo', callback);
me.stop_watching('repo', callback);
me.get_issues(params, callback);

// organization data
var org = octonode.Organization('bulletjs');

org.update(dict_with_update_properties, callback);
org.add_member('user', 'team', callback);
org.remove_member('user', callback);
org.get_public_members(callback);
org.is_public_member('user', callback);
org.make_member_public('user', callback);
org.conceal_member('user', callback);

org.get_team('team', callback);
org.create_team({name:'', repo_names:'', permission:''}, callback);
org.edit_team({name:'', permission:''}, callback);
org.delete_team('name', callback);
org.get_team_members('team', callback);
org.get_team_member('team', 'user', callback);
org.remove_member_from_team('user', 'team', callback);
org.get_repositories(callback);
org.create_repository({name: ''}, callback);
org.get_team_repositories('team', callback);
org.get_team_repository('team', 'name', callback);
org.add_team_repository('team', 'name', callback);
org.remove_team_repository('team', 'name', callback);

var repo = octonode.Repository('pksunkara/octonode');

repo.update({name: ''}, callback);

// collaborator information
repo.add_collaborator('name', callback);
repo.remove_collaborator('name', callback);

// commit data
repo.get_commit('sha-id', callback);
repo.get_all_comments(callback);
repo.get_commit_comments('SHA ID', callback);
repo.comment_on_commit({body: '', commit_id: '', line: '', path: '', position: ''}, callback);
repo.get_single_comment('comment id', callback);
repo.edit_single_comment('comment id', callback);
repo.delete_single_comment('comment id', callback);

// downloads
repo.get_downloads(callback);
repo.get_download(callback);
repo.create_download({name: ''}, 'filepath', callback);
repo.delete_download(callback);

// keys
repo.get_deploy_keys(callback);
repo.get_deploy_key('id', callback);
repo.create_deploy_key({title: '', key: ''}, callback);
repo.edit_deploy_key({title: '', key: ''}, callback);
repo.delete_deploy_key('id', callback);

// watcher data
repo.get_watchers(callback);

// pull requests
repo.get_all_pull_request_comments(callback);
repo.get_pull_request_comment('id', callback);
repo.create_pull_request_comment('id', {body:'', commit_id:'', path:'', position:''}, callback);
repo.reply_to_pull_request_comment('id', 'body', callback);
repo.edit_pull_request_comment('id', 'body', callback);
repo.delete_pull_request_comment('id', callback);
repo.get_issues(params, callback);
repo.get_issue('id', callback);
repo.create_issue({title: ''}, callback);
repo.edit_issue({title: ''}, callback);
repo.get_issue_comments('issue', callback);
repo.get_issue_comment('id', callback);
repo.create_issue_comment('id', 'comment', callback);
repo.edit_issue_comment('id', 'comment', callback);
repo.delete_issue_comment('id', callback);
repo.get_issue_events('id', callback);
repo.get_events(callback);
repo.get_event('id', callback);
repo.get_labels(callback);
repo.get_label('id', callback);
repo.create_label('name', 'color', callback);
repo.edit_label('name', 'color', callback);
repo.delete_label('id', callback);
repo.get_issue_labels('issue', callback);
repo.add_labels_to_issue('issue', ['label1', 'label2'], callback);
repo.remove_label_from_issue('issue', 'labelid', callback);
repo.set_labels_for_issue('issue', ['label1', 'label2'], callback);
repo.remove_all_labels_from_issue('issue', callback);
repo.get_labels_for_milestone_issues('milestone', callback);
repo.get_milestones(callback);
repo.get_milestone('id', callback);
repo.create_milestone('title', callback);
repo.edit_milestone('title', callback);
repo.delete_milestone('id', callback);

// raw git access
repo.create_blob('content', 'encoding', callback);
repo.get_commit('sha-id', callback);
repo.create_commit('message', 'tree', [parents], callback);
repo.get_reference('ref', callback);
repo.get_all_references(callback);
repo.create_reference('ref', 'sha', callback);
repo.update_reference('ref', 'sha', force, callback);
```

__I accept pull requests and guarantee a reply back within a day__

## License
MIT/X11

## Bug Reports
Report [here](http://github.com/pksunkara/octonode/issues). __Guaranteed reply within a day__.

## Contact
Pavan Kumar Sunkara (pavan.sss1991@gmail.com)

Follow me on [github](https://github.com/users/follow?target=pksunkara), [twitter](http://twitter.com/pksunkara)
