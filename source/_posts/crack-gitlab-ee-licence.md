---
title: gitlab-ee 企业版自签许可证license
date: 2019-12-23 11:27:40
author: "Mayer Shi"
tags: ["gitlab"]
categories: ["DevOps"]
---

本篇文章主要是研究gitlab-ee的licence签发方式，仅供大家学习参考使用。请尊重软件开发者成果，支持正版。

<!--more-->

## 背景

最近帮一个朋友推动开发测试一体化从DevOps转向GitOps。对于什么是GitOps，有时间搞个专题博客来讲讲这个概念以及最佳实践。总之在推进GitOps的时候需要，需要一些gitlab-ee 的高级特性。

### 1. 安装ruby环境以及gem包管理工具

- 由于我的电脑是mbp,所以自带ruby开发环境,无需安装。

- 安装相关ruby包依赖`gitlab`、`gitlab-license`、`openssl`

```bash
sudo gem install gitlab
sudo gem install gitlab-license
sudo gem install openssl
```

### 2. 编写创建license的ruby脚本，并生成license文件


- 创建脚本文件


```bash
vim createlicense.rb
```


- 文件内容如下

```ruby
require 'openssl'
require 'gitlab/license'

# Generate a key pair. You should do this only once.
key_pair = OpenSSL::PKey::RSA.generate(2048)

# Write it to a file to use in the license generation application.
File.open("license_key", "w") { |f| f.write(key_pair.to_pem) }

# Extract the public key.
public_key = key_pair.public_key
# Write it to a file to ship along with the main application.
File.open("license_key.pub", "w") { |f| f.write(public_key.to_pem) }

# In the license generation application, load the private key from a file.
private_key = OpenSSL::PKey::RSA.new File.read("license_key")
Gitlab::License.encryption_key = private_key

# Build a new license.
license = Gitlab::License.new

# License information to be rendered as a table in the admin panel.
# E.g.: "This instance of GitLab Enterprise Edition is licensed to:"
# Specific keys don't matter, but there needs to be at least one.
license.licensee = {
  "Name"    => "tester",
  "Company" => "Google Inc",
  "Email"   => "test@163.com"
}

# The date the license starts. 
# Required.
license.starts_at         = Date.new(2019, 4, 24) # license 开始生效时间
# The date the license expires. 
# Not required, to allow lifetime licenses.
license.expires_at        = Date.new(2026, 4, 23) # license 到期时间

# The below dates are hardcoded in the license so that you can play with the
# period after which there are "repercussions" to license expiration.

# The date admins will be notified about the license's pending expiration. 
# Not required.
license.notify_admins_at  = Date.new(2026, 3, 23) # license 管理员过期提醒时间

# The date regular users will be notified about the license's pending expiration.
# Not required.
license.notify_users_at   = Date.new(2026, 3, 23) # license 普通用户过期提醒时间

# The date "changes" like code pushes, issue or merge request creation 
# or modification and project creation will be blocked.
# Not required.
license.block_changes_at  = Date.new(2026, 5, 7) 

# Restrictions bundled with this license.
# Not required, to allow unlimited-user licenses for things like educational organizations.
license.restrictions  = {
  # The maximum allowed number of active users.
  # Not required.
  active_user_count: 10000  # license 人数配额

  # We don't currently have any other restrictions, but we might in the future.
}

puts "License:"
puts license

# Export the license, which encrypts and encodes it.
data = license.export

puts "Exported license:"
puts data

# Write the license to a file to send to a customer.
File.open("GitLabBV.gitlab-license", "w") { |f| f.write(data) }


# In the customer's application, load the public key from a file.
public_key = OpenSSL::PKey::RSA.new File.read("license_key.pub")
Gitlab::License.encryption_key = public_key

# Read the license from a file.
data = File.read("GitLabBV.gitlab-license")  # 生成license存储文件名

# Import the license, which decodes and decrypts it.
$license = Gitlab::License.import(data)

puts "Imported license:"
puts $license

# Quit if the license is invalid
unless $license
  raise "The license is invalid."
end

# Quit if the active user count exceeds the allowed amount:
if $license.restricted?(:active_user_count)
  active_user_count = 1000
  if active_user_count > $license.restrictions[:active_user_count]
    raise "The active user count exceeds the allowed amount!"
  end
end

# Show admins a message if the license is about to expire.
if $license.notify_admins?
  puts "The license is due to expire on #{$license.expires_at}."
end

# Show users a message if the license is about to expire.
if $license.notify_users?
  puts "The license is due to expire on #{$license.expires_at}."
end

# Block pushes when the license expired two weeks ago.
module Gitlab
  class GitAccess
    # ...
    def check(cmd, changes = nil)
      if $license.block_changes?
        return build_status_object(false, "License expired")
      end

      # Do other Git access verification
      # ...
    end
    # ...
  end
end

# Show information about the license in the admin panel.
puts "This instance of GitLab Enterprise Edition is licensed to:"
$license.licensee.each do |key, value|
  puts "#{key}: #{value}"
end

if $license.expired?
  puts "The license expired on #{$license.expires_at}"
elsif $license.will_expire?
  puts "The license will expire on #{$license.expires_at}"
else
  puts "The license will never expire."
end
```

- 执行以上license脚本文件，生成三个文件,

```bash
ruby createlicense.rb  # 执行脚本生成如下内容

License:
#<Gitlab::License:0x00007fd08691eca0>
Exported license:
eyJkYXRhIjoiWVo0VEIraWJQai8zUDhWRi9OK2Y3d2JXcG1ucVZGbXhUamtP
S0QyY01BSG9XYlBLRlh0QUcvQ1UzMm9EXG5tU1RSd0pqUmlRT2hOOC9KOWJi
Yk9mZ0krUmt5aWd2WnBNdGYydVZsUTFEemhFSU1jZWk5VjdtTWJycS9cbk9I
V3BmMjR3TFFmcXhQdHVvNVFDbVd0Z2Njc2lNUXhPUzVxUTN0YkRvRDRhYTk2
OVQvbjN4clc1RkNHZlxuUXU0TnZ1OHhqcjlZMTJIZk5yaVd3a2ZLZTBqZitU
cExvem9GTk9QY2d5R3hGdEtkNGVUVzdpa0tQRUNCXG5IRmZYVlVydHdBU3Fm
ei96WFNvaHRGVTFLWW1USkxMOFQ5eW5PTFdpQ1gycXRIYkF1T0hLV0N0bi9W
dE5cbjViZ3VRd05xd2hSOHlNanB6SFhNNC9OemZMVkRVQ3ZTTndVZWR6Q2Q1
Q3hrcE9BOEU1cFN5aXRiSTBkZ1xucTJ1ekJnSHNtdlVVdE5mbjFPUWtTc0FS
dkE2QUxUdGs1ZXZZN3Z6SDJ5N3FMZz09XG4iLCJrZXkiOiJnZ200N1hvRnkr
OTRYb09wcFh5akE1VVo0NTU5REhxeWhVVHMzVWV1dDhRRDRzZGpReVZwRGVp
QkFuajZcblgvL2RvYnM5QVRyYlRZa0V0SEJadUN5bGR5dUhlbEhQSHdJNUxS
RXgyeUpkb2NBRnRVVTNlTjdKcm9ZZ1xuVGJPaXk5c1E4eldNMWhZYWlWWDAy
eDdpTGx6eHA3eDJVSWJpRkZZd2J1dmZXeGtiMk10dnVQdFdsOUt0XG5mcW91
b0dYN0ZZclV1d3NWOGVNNTcyeS90elpMNFBLMFZvTE5vN3d6eEdveW1FbUNP
Sm9kYXVNR0IydjNcbkVybnFac2xsdlo1ZThnNDFKMElLclhLZ1lEK2J3WUR1
a1hqcHk4OU5GZHVaaGgzK3V5ZFNROWJwSC9wYVxuZjA4YUUwUzBkSzhyeTU2
SzVnbEVweFRtc09SZlBTUmhSSVhnalZsYVB3PT1cbiIsIml2IjoiN1VLSTh0
UXo0aFZ6bGV5QW9kSWFxQT09XG4ifQ==
Imported license:
#<Gitlab::License:0x00007fd086926f40>
This instance of GitLab Enterprise Edition is licensed to:
Name: test
Company: Google Inc
Email: test@163.com
The license will expire on 2026-04-23

ls . # 查看当前目录下文件
GitLabBV.gitlab-license createlicense.rb        license_key             license_key.pub
```

### 3. 替换公钥以及激活license

- 将createlicense.rb脚本文件生成的license_key.pub公钥内容替换到gitlab的`/opt/gitlab/embedded/service/gitlab-rails/.license_encryption_key.pub`中。然后重启gitlab，`gitlab-ctl restart`.


- 重启完毕后，将脚本生成的GitLabBV.gitlab-license文件，导入gitlab激活页面即可激活了。


### 4. 修改gitlab-ee的等级为ULTIMATE_PLAN


- 修改gitlab的文件


`vim /opt/gitlab/embedded/service/gitlab-rails/ee/app/models/license.rb`
```ruby
   def plan
     restricted_attr(:plan).presence || STARTER_PLAN  ## 将STARTER_PLAN 修改为 ULTIMATE_PLAN
   end
```


### 小问题


>- 就是生成的license虽然可以激活，但是license页面老是报500错误。这个得看下后台日志排查下即可。

