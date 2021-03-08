# Admin-API

As of version 4.0 a new administration interface and API _leihs_ has been added
to leihs. This new interface and the corresponding API can be accessed via the
path `/admin/`.



## API

The newly created `/admin/*` routes are also accessible via an API. In fact the
visible front-end communicates exclusively over this new API with the server.
Exploring the new API can be done by inspecting the communication of the
web-browser with the server. Experienced developers will know how to proceed to
this end. See also the section "Example Applications" below.


### Managed Entities and Resources

A few resources under `/admin/` link to older implementations which are not
designed to be driven automatically. But the more useful ones, like
`/admin/users/` and `/admin/groups/` have been ported.

As mentioned above the ported resources can be easily inspected with the
developer tools of a browser.


#### Updating Group Membership

The resource `/admin/groups/:id/users/` used with the `PUT` updates the all the
members of a group with one single request. This combination is not available
via the front end but it is very useful for API clients.

A request with the payload e.g.

```
{
   "ids":[ "85cab111-4790-5b83-9c75-343e82037d41",
           "c0777d74-668b-5e01-abb5-f8277baa0ea8"]
}
```

sets the target group to have **exactly** the two users with the respective ids
as members irregardless of the membership state before the request.

Supplying the previously supported `org_ids` and `emails` keys in **leihs >=
6.0** will result in an unprocessable entity 422 response.


### API Token-Authentication

For automated purposes the session cookies would be a poor way to authenticate
with. _leihs_ as of version 4.0 features API-token authentication which are
meant to be used for automated purposes.

An API-token is always bound to a user. Access to various parts of the
application are restricted by the access rights the user has who is owner of
the token.

For the time being tokens can be managed via the path
`/my/user/me/api-tokens/`.

To use a token with an API call it is added as an HTTP-header with
the name `Authorization` and value `Token ${the_token}`. See also the
specifications, e.g. https://github.com/leihs/leihs-admin/blob/05a23dfef3f7fb4cf80a4254c7485611ba7cc484/spec/features/groups/manage-group-users_batch-api_spec.rb#L16
or the "Example Applications" below.


## Example Applications using the API

*  [ZHdK User and Group Sync for _leihs_](https://github.com/leihs/zhdk-sync)
