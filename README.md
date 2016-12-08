About Varnish
=============
Varnish is a very fast reverse-proxy system which serves static 
files and anonymous page views based on the previously processed
requests.

Nexteuropa Varnish
==================
The Nexteuropa Varnish module provides functionality which allows to
send customized HTTP request to the Varnish server based on the
configured 'purge rules'.
The main purpose of those requests is to invalidate the Varnish cache to
reflect recently published content changes.

## Requirements
This feature can be enabled only with the support of the QA/Maintenance
team.
The following environment specific variables have to be configured
before enabling the feature:
- 'nexteuropa_varnish_request_user' 
- 'nexteuropa_varnish_request_password'
- 'nexteuropa_varnish_http_targets',
- 'nexteuropa_varnish_tag',
- 'nexteuropa_varnish_request_method'

In order to enable the feature make sure that above variables are set
and if so then go to the `admin/structure/feature-set` page,
select the 'Rule-based web frontend cache purging' feature 
and click on the 'Validate' button.

Nexteuropa Varnish provides a 'Administer frontend cache purge rules'
permission which allows to create and maintain 'purge rules'.

## Custom entity - 'Purge rule'
Nexteuropa Varnish provides an additional custom entity type:
- Purge rule - machine name: `nexteuropa_varnish_cache_purge_rule`

It allows to create and maintain a set of rules which are responsible
for sending customized HTTP requests to the Varnish server in order to
invalidate specific frontend cache items.

To add and maintain purge rules go to the following url:
`admin/config/frontend_cache_purge_rules`

## How to add and maintain 'Purge rule'
To add new cache purge rule you can expand the **Configuration -> Cache purge rules** menu
and click on the **'Add cache purge rule'** link.
You will be redirected to the **'Add cache purge rule'** form.

In the first step you need to choose a content type for which the new rule will be applied.
After picking the content type the next step is to choose the purge target.

The 'Paths of the node the action is performed on' option will perform a purge
on any path that belongs to the specifiec content type.

The 'A specific list of paths' option allows to define a set of paths
that are going to be purged.
The textfield allows some wildcards characters:
- '*' (asterisk) matches any number of any characters including none
- '?' (question mark) matches any single character

After setting up a rule you need to submit it by clicking the **'Save'** button.

After the creation of a new rule you will be redirected to the page with the list of rules.
From that page you can use option to add a new rule or edit, delete existing rules.

## Purge rules logic
The Nexteuropa Varnish provides hardcoded logic for triggering
configured rules. Current version implements two workflow cases for:
- content types moderated via the workbench moderation module
- content types without additional moderation (default Drupal settings)

### Content moderated via the workbench moderation module
For the content types which are controlled via the workbench moderation
module, created purge rules will be triggered in the following cases:
- when a given content has a workflow state change to 'Published'
- when a given content has a workflow state change from 'Published' to any other

### Content without moderation
For the content types which are not moderated (default Drupal content
type with two states: published and unpublished), created purge rules
will be triggered in the following cases:
- when a node of the given content type is created and saved with the 'Publish' state
- when a published node of the given content type is updated

## Tests and custom Behat Feature Context
The Nexteuropa Varnish provides complete a Behat test suite and additional
Feature Context located in the FrontendCacheContext class.

Tests are performed against a mocked HTTP server. The only difference is that
the mocked HTTP server doesn't support 'PURGE' method and uses
the 'POST' method instead.

You can find the Behat scenarios in the frontend_cache_purge.feature file
located under the test folder.

## Developer's notes
Nexteuropa Varnish uses the https://www.drupal.org/project/chr module
which overrides the default `drupal_http_request()` function.

A custom patch was created for this specific feature
The patch can be found [here](https://www.drupal.org/files/issues/chr-purge-2825701-2.patch)

The patch adds the 'PURGE' HTTP method, which is commonly used by systems such
as Varnish, Squid and SAAS CDNs like Fastly to clear cached versions of
certain paths.

All of HTTP requests are send by the `_nexteuropa_varnish_purge_paths()`
function.
