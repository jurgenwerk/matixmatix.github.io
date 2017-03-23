---
category: posts
mainTopic: 'programming'
layout: single
title: "List and download Dropbox files with Ruby"
excerpt: "How to use Ruby Dropbox SDK to download files."
---

If you find yourself in a need for downloading contents from Dropbox (or whole folders), a sane and quick way would be
to visit their web interface and select to download (compressed) files.

But what about when you don't have a password to access the interface, but you do have an API access token?

I put together a small script to help you with that. Here it goes.

```ruby
require "fileutils"
require "dropbox_sdk" # dropbox-sdk-ruby gem

client = DropboxClient.new("access_token")

# Recursively traverse the contents in dir_path
def list_files(client, dir_path)
  metadata = client.metadata(dir_path)
  metadata["contents"].flat_map do |f|
    if f["is_dir"]
      list_files(client, f["path"])
    else
      f["path"]
    end
  end
end

# A folder on your local machine where contents will be downloaded to
dump_folder = "dropbox_dump"

# List files for the root path ('/')
file_paths = list_files(client, "/")
```

Now we have the paths of all the files in the folder we provided (root folder). Let's download them all.

```ruby
def download_file(client, dump_folder, file_path)
  full_file_path = dump_folder + file_path
  if File.exist?(full_file_path)
    puts "File #{full_file_path} already exists"
    return
  end

  contents, metadata = client.get_file_and_metadata(file_path)
  full_folder_path = File.dirname(full_file_path)
  FileUtils.mkdir_p(full_folder_path)
  open(full_file_path, 'wb') { |f| f.puts contents }
end

# Download every file in file_paths to specified local dump folder
file_paths.each_with_index do |file_path, index|
  puts "Downloading #{file_path} ..."
  download_file(client, dump_folder, file_path)
  puts "#{file_paths.length - index + 1} to go"
end
```
