# HTTP Directory Index Resource

A Concourse resource for getting releases from HTTP directory indexes.

This will work for source code releases where a zip/tar is published to a static http page with a list of all previous releases (eg for [pcre](https://ftp.pcre.org/pub/pcre/))

## Source Configuration

```yaml
resource_types:
- name: http-directory-index
  type: docker-image
  source:
    repository: shreddedbacon/http-directory-index

resources:
- name: pcre
  type: http-directory-index
  source:
    url: https://ftp.pcre.org/pub/pcre/
    file_prefix: pcre-
    file_suffix:
    file_regex: ([0-9]+\\.)?([0-9]+)?
    file_extension: .tar.gz
    file_reverse: false
```

* `url` is the url for the page that has the resources you want to check
* `file_prefix` is what is infront of the version numbering scheme. This should not really be left blank
* `file_suffix` is what is after the version numbering scheme. This can be left blank
* `file_regex` is used to match the versioning schme used by the release resource. Craft your own regex to match your numbering scheme.
* `file_extension` the extension of the file you want as a resource. Defaults to `.tar.gz`
* `file_reverse` if the directory index resources are not in ascending order, this will reverse the list so they are. Defaults to false

The check uses grep only-match to the following pattern in the output of the http page `{file_prefix}{file_regex}{file_suffix}{file_extension}`

It assumes that the files are located in the same directory as the url. If the url is `https://ftp.pcre.org/pub/pcre/`, then the file should be available here `https://ftp.pcre.org/pub/pcre/pcre-8.13.tar.gz`.

The `in` task will only download the file if it exists in the source url.

## Behavior

### `check`: Check for release tags

Checks if there are new versions of the source. You can query the resource for specific versions using `fly -t <target> check-resource -r <pipeline>/<resource_name> --from version:<version>`

### `in`: Download a version

Places the following files in the destination:

* `source{file_extension}`: The source file from the release page
* `url`: A file containing the URL of the file that was downloaded
* `version`: The version of the resource

### `out`: Not implemented

Does nothing

## Example

```yaml
jobs:
- name: get-pcre-resource
  plan:
  - get: pcre
    trigger: true
```
