<script>
(function test() {
    const results = document.getElementsByClassName('see-result')
    const res = Array.from(results)
    if (res.length > 0) {
        res.map(el => {
            console.log(el)
            el.addEventListener('click', e => {
                this.toggleCodeBlock()
            })
        })
    }
})()

function toggleCodeBlock(e) {
    console.log('toggle')
    const el = e.target
    const sibling = el.nextElementSibling
    const style = sibling.getAttribute('style')
    if (style === 'display:none') {
        el.nextElementSibling.setAttribute('style', 'display:block')
    } else if (style === 'display:block') {
        el.nextElementSibling.setAttribute('style', 'display:none')
    }
}
</script>

# API

## Installation

Follow the [installation instructions](/docs/server/install) for Sourcegraph Server.

## API overview

The Sourcegraph API has two components: the GraphQL API and the Language Server Protocol Gateway (LSP Gateway).

At a high level, the GraphQL API supports the following types of queries:

- Full-text and regex code search
- Rich git-level metadata, including commits, branches, blame information, and file tree data
- Raw git commands like `git log --author=John` and `git blame -- app.py`
- Dependency graph queries ("Which repositories use library X?")
- Repository and user metadata

The LSP Gateway uses the [Language Server Protocol](https://github.com/Microsoft/language-server-protocol) to enable Code Intelligence features like jump-to-definition, find-references, and auto-completion. The LSP community has created [open-source language servers](http://langserver.org/) for nearly every major language. The LSP Gateway sits in front of these language servers and provides a single endpoint for developer tools and services wishing to offer Code Intelligence for many languages at once.

The Sourcegraph API supports Sourcegraph Server and Sourcegraph Editor. It also backs a variety of first- and third-party extensions, integrations, and plugins. The most popular public extension is the [Sourcegraph Chrome Extension](https://chrome.google.com/webstore/detail/sourcegraph-for-github/dgjhfomjieaadpoljlnidmbgkdffpack), which adds Code Intelligence to diffs and code views on GitHub.com. Integrations also exist for GitLab, Bitbucket Server, Bitbucket.org, Phabricator, and a range of other code hosts, review tools, and editors.

<em>Note: these public docs refer to the latest version of the Sourcegraph API, which changes each month. Paid Sourcegraph Server instances support API versioning.</em>

### Explore the API

- [Interactive GraphiQL query explorer](https://sourcegraph.com/.api/graphql)
- <a href="https://about.sourcegraph.com/docs/server/schema/" target="_top">Reference documentation</a>


### Example queries

Here are a few examples of the types of queries you can make with the Sourcegraph API. You can try these out yourself using the [GraphiQL explorer](https://sourcegraph.com/.api/graphql).

### Code search across all your repositories, with regex support

```bash
curl 'https://sourcegraph.com/.api/graphql' -d '{"query":"{root{searchRepos(query:{pattern:\"route\",isRegExp:false,isWordMatch:false,isCaseSensitive:false,fileMatchLimit:10},repositories:[{repo:\"github.com/gorilla/mux\",rev:\"master\"},{repo:\"github.com/gorilla/pat\",rev:\"master\"}]){results{resource}}}}","variables":null,"operationName":null}'
```
```graphql
{
  root {
    searchRepos(
      query: {
        pattern: "route", isRegExp: false, isWordMatch: false, isCaseSensitive: false, fileMatchLimit: 10
      },
      repositories: [
        {repo: "github.com/gorilla/mux", rev: "master"},
        {repo: "github.com/gorilla/pat", rev: "master"}
      ]
    ) {
      results {
        resource
      }
    }
  }
}
```

<div class="flex bg-light-10 pa2 see-result b pointer" style='display:flex;padding:1rem;background:#e4e9f1;margin:auto;margin-top:1rem;margin-bottom:1rem;width:95%;' onclick='toggleCodeBlock(event)'>
	<svg viewBox="0 0 64 64" class="icon icon-caret-down h15 w15" style='height:1.5rem;width:1.5rem'>
		<title>Icons 100</title>
		<path d="M18.4 28.6L32 42.2l13.6-13.6H18.4z"></path>
	</svg>
	<div>Result</div>
</div>
<pre class="highlight json tab-json" style="display:none">

<code>
{
  "data": {
    "root": {
      "searchRepos": {
        "results": [
          {
            "resource": "git://github.com/gorilla/pat?cf955c3d1f2c27ee96f93e9738085c762ff5f49d#pat_test.go"
          },
          {
            "resource": "git://github.com/gorilla/pat?cf955c3d1f2c27ee96f93e9738085c762ff5f49d#pat.go"
          },
          {
            "resource": "git://github.com/gorilla/pat?cf955c3d1f2c27ee96f93e9738085c762ff5f49d#doc.go"
          },
          {
            "resource": "git://github.com/gorilla/pat?cf955c3d1f2c27ee96f93e9738085c762ff5f49d#README.md"
          },
          {
            "resource": "git://github.com/gorilla/mux?3f19343c7d9ce75569b952758bd236af94956061#route.go"
          },
          {
            "resource": "git://github.com/gorilla/mux?3f19343c7d9ce75569b952758bd236af94956061#regexp.go"
          },
          {
            "resource": "git://github.com/gorilla/mux?3f19343c7d9ce75569b952758bd236af94956061#old_test.go"
          },
          {
            "resource": "git://github.com/gorilla/mux?3f19343c7d9ce75569b952758bd236af94956061#mux_test.go"
          },
          {
            "resource": "git://github.com/gorilla/mux?3f19343c7d9ce75569b952758bd236af94956061#mux.go"
          },
          {
            "resource": "git://github.com/gorilla/mux?3f19343c7d9ce75569b952758bd236af94956061#doc.go"
          }
        ]
      }
    }
  }
}
</code>

</pre>

### Find all dependents of a package
```bash
curl 'https://sourcegraph.com/.api/graphql' -d '{"query":"{root{dependents(lang:\"python\",name:\"werkzeug\",limit:10){repo{uri}}}}","variables":null,"operationName":null}'
```
```graphql
{
  root {
    dependents(lang: "python", name: "werkzeug", limit: 10) {
      repo {
        uri
      }
    }
  }
}
```

<div class="flex bg-light-10 pa2 see-result b pointer" style='display:flex;padding:1rem;background:#e4e9f1;margin:auto;margin-top:1rem;margin-bottom:1rem;width:95%;' onclick='toggleCodeBlock(event)'>
	<svg viewBox="0 0 64 64" class="icon icon-caret-down h15 w15" style='height:1.5rem;width:1.5rem'>
		<title>Icons 100</title>
		<path d="M18.4 28.6L32 42.2l13.6-13.6H18.4z"></path>
	</svg>
	<div>Result</div>
</div>
<pre class="highlight json tab-json" style="display:none">

<code>
{
  "data": {
    "root": {
      "dependents": [
        {
          "repo": {
            "uri": "github.com/Modulus/Flask-RestfulDemo"
          }
        },
        {
          "repo": {
            "uri": "github.com/inveniosoftware/invenio-github"
          }
        },
        {
          "repo": {
            "uri": "github.com/Dav1dde/glad-web"
          }
        },
        {
          "repo": {
            "uri": "github.com/brutasse/graphite-api"
          }
        },
        {
          "repo": {
            "uri": "github.com/kennethreitz/flask-sockets"
          }
        },
        {
          "repo": {
            "uri": "github.com/xbynet/mdwiki"
          }
        },
        {
          "repo": {
            "uri": "github.com/regaleagle/SQUID"
          }
        },
        {
          "repo": {
            "uri": "github.com/swindon-rs/swindon"
          }
        },
        {
          "repo": {
            "uri": "github.com/superhuahua/xunfengES"
          }
        },
        {
          "repo": {
            "uri": "github.com/xiechao06/Flask-DataBrowser"
          }
        }
      ]
    }
  }
}
</code>

</pre>

### Query rich git metadata
```bash
curl 'https://sourcegraph.com/.api/graphql' -d '{"query":"{root{repositories(query:\"gorilla/websocket\"){uri,description,language,pushedAt,defaultBranch,branches,tags,revState(rev:\"master\"){commit{sha1,tree(path:\"examples/autobahn\"){directories{name}files{name,blame(startLine:1,endLine:2){author{person{email}}}}}languages}}}}}","variables":null,"operationName":null}'
```
```graphql
{
  root {
    repositories(query: "gorilla/websocket") {
      uri
      description
      language
      pushedAt
      defaultBranch
      branches
      tags
      revState(rev: "master") {
        commit {
          sha1
          tree(path: "examples/autobahn") {
            directories {
              name
            }
            files {
              name
              blame(startLine: 1, endLine: 2) {
                author {
                  person {
                    email
                  }
                }
              }
            }
          }
          languages
        }
      }
    }
  }
}
```

<div class="flex bg-light-10 pa2 see-result b pointer" style='display:flex;padding:1rem;background:#e4e9f1;margin:auto;margin-top:1rem;margin-bottom:1rem;width:95%;' onclick='toggleCodeBlock(event)'>
	<svg viewBox="0 0 64 64" class="icon icon-caret-down h15 w15" style='height:1.5rem;width:1.5rem'>
		<title>Icons 100</title>
		<path d="M18.4 28.6L32 42.2l13.6-13.6H18.4z"></path>
	</svg>
	<div>Result</div>
</div>

<pre class="highlight json tab-json" style="display:none">

<code>
{
 "data": {
	"root": {
	"repositories": [
		{
		"uri": "github.com/gorilla/websocket",
		"description": "A WebSocket implementation for Go.",
		"language": "Go",
		"pushedAt": "2017-09-26 23:33:36 +0000 UTC",
		"defaultBranch": "master",
		"branches": [
			"master"
		],
		"tags": [
			"v1.0.0",
			"v1.1.0",
			"v1.2.0"
		],
		"revState": {
			"commit": {
			"sha1": "4201258b820c74ac8e6922fc9e6b52f71fe46f8d",
			"tree": {
				"directories": [],
				"files": [
				{
					"name": "README.md",
					"blame": [
					{
						"author": {
						"person": {
							"email": "gary@beagledreams.com"
						}
						}
					},
					{
						"author": {
						"person": {
							"email": "gary@beagledreams.com"
						}
						}
					}
					]
				},
				{
					"name": "fuzzingclient.json",
					"blame": [
					{
						"author": {
						"person": {
							"email": "gary@beagledreams.com"
						}
						}
					}
					]
				},
				{
					"name": "server.go",
					"blame": [
					{
						"author": {
						"person": {
							"email": "gary@beagledreams.com"
						}
						}
					}
					]
				}
				]
			},
			"languages": [
				"Go",
				"Markdown",
				"HTML",
				"JSON",
				"YAML"
			]
			}
		}
		}
	]
	}
}
}
</code>
</pre>


### Raw git data
```bash
curl 'https://sourcegraph.com/.api/graphql' -d '{"query":"{root{repository(uri:\"github.com/gorilla/mux\"){gitCmdRaw(params:[\"log\",\"--author=Chris\"])}}}","variables":null,"operationName":null}'
```
```graphql
{
  root {
    repository(uri:"github.com/gorilla/mux"){
      gitCmdRaw(params: ["log", "--author=Chris"])
    }
  }
}
```

<div class="flex bg-light-10 pa2 see-result b pointer" style='display:flex;padding:1rem;background:#e4e9f1;margin:auto;margin-top:1rem;margin-bottom:1rem;width:95%;' onclick='toggleCodeBlock(event)'>
	<svg viewBox="0 0 64 64" class="icon icon-caret-down h15 w15" style='height:1.5rem;width:1.5rem'>
		<title>Icons 100</title>
		<path d="M18.4 28.6L32 42.2l13.6-13.6H18.4z"></path>
	</svg>
	<div>Result</div>
</div>
<pre class="highlight json tab-json" style="display:none">

<code>
{
  "data": {
    "root": {
      "repository": {
        "gitCmdRaw": "commit ac112f7d75a0714af1bd86ab17749b31f7809640\nAuthor: Chris Hines <github@cs-guy.com>\nDate:   Mon Jul 3 11:07:09 2017 -0400\n\n    Prefer scheme on child route when building URLs.\n\ncommit 37b3a6cace5ebe92c9b1494fab04d5e7767b271b\nAuthor: Chris Hines <github@cs-guy.com>\nDate:   Fri Jun 30 10:51:05 2017 -0400\n\n    Use scheme from parent router when building URLs.\n\ncommit 18fca31550181693b3a834a15b74b564b3605876\nAuthor: Chris Hines <github@cs-guy.com>\nDate:   Tue May 30 15:55:47 2017 -0400\n\n    Add test and fix for escaped query values.\n    \n    Reproduces and fixes #238.\n\ncommit c7a138dbc1778cafd4aaddb7ce7975a5f3649a49\nAuthor: Chris Hines <github@cs-guy.com>\nDate:   Tue May 30 15:53:16 2017 -0400\n\n    Update docs.\n\ncommit bcd8bc72b08df0f70df986b97f95590779502d31\nAuthor: Chris Hines <github@cs-guy.com>\nDate:   Sun May 21 00:50:13 2017 -0400\n\n    Support building URLs with non-http schemes. (#260)\n    \n    * Move misplaced tests and fix comments.\n    \n    * Support building URLs with non-http schemes.\n    \n    - Capture first scheme configured for a route for use when building\n      URLs.\n    - Add new Route.URLScheme method similar to URLHost and URLPath.\n    - Update Route.URLHost and Route.URL to use the captured scheme if\n      present.\n    \n    * Remove Route.URLScheme method.\n    \n    * Remove UTF-8 BOM.\n\ncommit 04a79835ae36db13cbcc39e8420082a48549a42a\nAuthor: Christopher Pfohl <Christopher.Pfohl@gmail.com>\nDate:   Thu Aug 29 12:05:40 2013 -0400\n\n    Add \"of\" like the rest of the function docstrings\n"
      }
    }
  }
}
</code>
</pre>
