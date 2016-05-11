---
  layout: post
  title: Using private git repos as NPM modules
  tags: npm 
  categories: 
---
One of the really cool, more obscure, features of npm is the ability to install git repos directly via the `npm install` command. 


## The problem
The problem is that installing packages this way, the ssh username will be saved in the package.json file. Basically, making this dependency useless for the rest of your team. 

`npm install -s git+ssh://<username>@<hostname>:<path>`

After installing a package via a command like the one above. Your dependency in package.json, instead of the common semver versioning number (^1.0.0.), would instead be the exact url used to install the package, username and all.

So, if that package.json gets commited, no one else on your team would be able to `npm install` the dependencies as they wouldn't have username's ssh credentials.

{% highlight json %}
  "dependencies":{
    "lodash": "^4.0.0",
    "privateModule": "git+ssh://dudemullet@server.com"  
  }
{% endhighlight %}
_Example package.json file with private git repo dependency_

## Help us ssh! (the solution)
The clue to solving this problem is in the url's protocol 'git+ssh'. Npm is basically using 2 different tools, git and ssh, to access the contents of your module. So, if we could access that module without typing our credentials in the url, problem solved!

Thankfully, ssh has a very helpful way of configuring it via an `~/.ssh/config` file. In it you can alias different hosts and typein specific configurations for each of these hosts.

{% highlight yaml %}
# This is a comment
Host server.com
    User foo
    HostName server.com
    IdentityFile ~/.ssh/foo
{% endhighlight %}
_Example ~/.ssh/config file_

With this last config file as an example, I could now type `ssh myreal.server.com` and it would try connecting to server.com using foo users private key. But the real money maker for our situation is that, we could also now type `npm install -s git+ssh://server.com:/path/to/privateModule.git` and our package.json would set the dependency field to:

{% highlight json %}
  "dependencies":{
    "lodash": "^4.0.0",
    "privateModule": "git+ssh://server.com:/path/to/privateModule.git"  
  }
{% endhighlight %}

Yay, no user specificity!

## The caveat
At this point you're either really happy this solved a *HUGE* problem you had, or you're kinda bummed you know what I'm gonna say next. *Any developer that uses this, needs to have the same alias for the same hostname in ~/.ssh/config*. Otherwise, ssh won't know what credentials to use for what server ¯\\_(ツ)_/¯. 

**Note**: this doesn not mean developers need to share private keys!


{% highlight yaml %}
# Dev1 config file
Host server.com
    User foo
    HostName server.com
    IdentityFile ~/.ssh/myIdentityFile
{% endhighlight %}

{% highlight yaml %}
# Dev2 config file
Host server.com
    User foo
    HostName server.com
    IdentityFile ~/.ssh/id_rsa

Host github.com
  User bar
  HostName github.com
  IdentityFile ~/.ssh/id_rsa
{% endhighlight %}

_Developers with different config files, only sharing the needed information to be able to install private modules from server.com_

## Versioning
One last important difference, is that we have to use a different versioning scheme. With regular npm registry modules, you could simply add it as the semver value to a packages json entry in the dependencies field.

{% highlight json %}
  dependencies:{ 
    "lodash": "^4.0.0"  
  }
{% endhighlight %}

But since with this method, the right-hand side will be a url, we need to append to that url a _commit-ish_ (I swear thats the legit parlance) code. And that means any branch name, commit hash or tag. Also important to note, we **cant** use a package.json's version number.

`npm install --save git+ssh://server.com:/path/to/privateModule.git#dev`
_installs whatever version of the module the dev branch points to_

`npm install --save git+ssh://server.com:/path/to/privateModule.git#v1.0.4`
_installs the version of the module pointed at by tag v1.0.4_

`npm install --save git+ssh://server.com:/path/to/privateModule.git#1b1d8f5`
_installs the version of the module at commit 1b1d8f5_

### info
- commit-ish [documentation](https://www.kernel.org/pub/software/scm/git/docs/gitrevisions.html#_specifying_revisions).
- npm install [documentation](https://docs.npmjs.com/cli/install)
