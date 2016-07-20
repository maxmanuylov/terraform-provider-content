# Terraform Content Provider

This is a plugin for HashiCorp [Terraform](https://terraform.io), which helps fetching some content from a remote URL, saving it (or another content) locally to a file, reusing variables with interpolated values.

## Usage

- Download the plugin from [Releases](https://github.com/maxmanuylov/terraform-provider-content/releases) page.
- [Install](https://terraform.io/docs/plugins/basics.html) it, or put into a directory with configuration files.
- Create a sample configuration file `terraform.tf`:
```
resource "content_by_url" "readme" {
  url = "https://raw.githubusercontent.com/maxmanuylov/terraform-provider-content/master/README.md"
}

resource "content_var" "readme_storage_path" {
  value = "${path.root}/readme_storage"
}

resource "content_dir" "readme_storage" {
  path = "${content_var.readme_storage_path.value}"
  permissions = "777"
}

resource "content_file" "readme" {
  path = "${content_dir.readme_storage.dir}/README.md"
  content = "${content_by_url.readme.content}"
  permissions = "644"
}

resource "content_command" "reconfigure" {
  trigger = "${content_var.readme_storage_path.value}"

  provisioner "local-exec" {
    command = "<Some command>"
  }
}
```
- Run:
```
$ terraform apply
```

All resource types can be used independently, so you can, for example, save some static content to the local file using just the "content_file" resource.

## The "content_by_url" resource type

Fetches content from the specified URL and provides it as a computed attribute.

### Mandatory Parameters
- `url` - content URL to fetch

### Computed Parameters
- `content` - fetched content

## The "content_dir" resource type

Manages (creates/removes/recreates) the local directory by the specified path. All parent directories are created as well if needed.

### Mandatory Parameters
- `path` - local directory path

### Optional Parameters
- `permissions` - string with octal directory permissions (e.g. "644")

### Computed Parameters
- `dir` - equal to `path` but is set _after_ the directory is created (so you can depend on this resource and be sure the directory already exists by the time you use it)

## The "content_file" resource type

Saves the specified content to the local file.

### Mandatory Parameters
- `path` - local file path
- `content` - content to save

### Optional Parameters
- `permissions` - string with octal file permissions (e.g. "644")

### Computed Parameters
- `file` - equal to `path` but is set _after_ the file is written (so you can depend on this resource and be sure the file already exists by the time you use it)

## The "content_var" resource type

Provides simple named variable. Unlike the built-in Terraform variables these variables can have interpolations in its values.

### Mandatory Parameters
- `value` - variable value

## The "content_command" resource type

Defines a resource that is got recreated every time the value of the `trigger` field changes. This is a convenient way of calling some provisioner on every `trigger` field change.

### Mandatory Parameters
- `trigger` - the value to monitor for changes
