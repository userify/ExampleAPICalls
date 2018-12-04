# ExampleAPICalls

Working with the Userify API is pretty simple with high-level tools like `curl` and `jq`. Some examples follow. Note that username and password can be either an API username and password (those are restricted to the post_shim API endpoint) or a regular username and password tunneled within HTTPS (TLS).

Nearly all of the company data for a particular user ID can be retrieved in a single API call called "my companies", which includes all of the companies, projects, and user ID's. We can then use `jq` to filter that down to just the data we're looking for.

In the following examples, we could optimize by just performing that my companies API call once and then reusing the data, but for simplicity and clarity we're explicitly re-calling it.

All Userify API endpoints begin with /api/userify/, but you may need to adjust your API endpoint if you are working with a local (self-hosted) server such as Userify Express or Userify Enterprise.

```
basepath=https://api.userify.com/api/userify/
alias userify_api="curl -su username:secret "
```

Let's start by retrieving a `company_id` for further calls. This `company_id` will be the first one returned if there are more than one. You can perform more complicated `jq` filtering if you have a specific one to work with.

### Reviwing the full `companies` data dump

```
userify_api $basepath/companies | jq .
```

### Choose a company ID from the user's companies:

```
company_id="$(userify_api $basepath/companies | jq -r .companies[0].id)"
```

### Choose a user ID from the company administrators:

```
user_id=$(userify_api $basepath/companies | jq -r .companies[0].admins[0])
```

### Create a new top-level project within this company

```
project_name="New Project"
new_project_id=$(userify_api -d "{\"name\":\"$project_name\"}" $basepath/project/company_id/$company_id \
  | jq -r .project_id)
```

### Create a new server group within this company

```
servergroup_name="Demo Servers"
servergroup_id=$(userify_api -d "{\"name\":\"$servergroup_name\"}" \
  $basepath/project/company_id/$company_id/parent_project_id/$new_project_id \
  | jq -r .project_id)
```

### Grant "root" (Linux sudo privileges)

This applies root/sudo to this user account for all servers in this server group:

```
userify_api -X PUT -d "{}" \
 $basepath/usergroup/company_id/$company_id/project_id/$servergroup_id/usergroup/linux_admins/user_id/$user_id
```

### Retrieve the API ID and API Key for the new server group:

```
userify_api $basepath/shim_installers/company_id/$company_id/project_id/$servergroup_id | jq -r .json | \
   jq  '.api_id + "," + .api_key'
```
