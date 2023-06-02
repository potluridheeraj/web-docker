---
title: Inspec Cheetsheet
keywords: Cheetsheet 
#summary: "DHEERAJ POTLURI"
sidebar: 
permalink: inspec_sheet
folder: Inspec
---
```shell
Controls
control 'sshd-8' do
  impact 0.6
  title 'Server: Configure the service port'
  desc 'Always specify which port the SSH server should listen.'
  desc 'rationale', 'This ensures that there are no unexpected settings' # Requires InSpec >=2.3.4
  tag 'ssh','sshd','openssh-server'
  tag cce: 'CCE-27072-8'
  ref 'NSA-RH6-STIG - Section 3.5.2.1', url: 'https://www.nsa.gov/ia/_files/os/redhat/rhel5-guide-i731.pdf'

  describe sshd_config do
    its('Port') { should cmp 22 }
  end
end
```
where

'sshd-8' is the name of the control  
impact, title, and desc define metadata that fully describes the importance of the control, its purpose, with a succinct and complete description  
desc when given only one argument it sets the default description. As of InSpec 2.3.4, when given 2 arguments (see: 'rationale') it will use the first argument as a header when rendering in Automate  
impact is a string, or numeric that measures the importance of the compliance results. Valid strings for impact are none, low, medium, high, and critical. The values are based off CVSS 3.0. A numeric value must be between 0.0 and 1.0. The value ranges are:  
0.0 to <0.01 these are controls with no impact, they only provide information  
0.01 to <0.4 these are controls with low impact  
0.4 to <0.7 these are controls with medium impact  
0.7 to <0.9 these are controls with high impact  
0.9 to 1.0 these are critical controls  
tag is optional meta-information with with key or key-value pairs  
ref is a reference to an external document  
describe is a block that contains at least one test. A control block must contain at least one describe block, but may contain as many as required  
sshd_config is an InSpec resource. For the full list of InSpec resources, see InSpec resource documentation  
its('Port') is the matcher; { should eq '22' } is the test. A describe block must contain at least one matcher, but may contain as many as required  
Resource properties  
You can use a string with dots to specify a nested attribute (i.e. an attribute of a property of the resource).  

Note:
its only supports universal matchers

its('last_changes.min') { should be < Date.today - 90 - Date.new(1970,1,1) }
its('min_days.uniq') { should eq [0] }
its('max_days.max') { should be < 90 }
its('warn_days.uniq.count') { should eq 1 }
its('warn_days.uniq.first') { should eq 7 }
its('expiry_dates.compact') { should be_empty }
its('ingest.first.processors.count') { should be >= 1 }
Matchers
The following InSpec-supported universal matchers are available:

be - make numeric comparisons
be_in - look for the property value in a list
cmp - general-use equality (try this first)
eq - type-specific equality
include - look for an expected value in a list-valued property
match - look for patterns in text using regular expressions
Examples:

should be_nil.or eq('None')
should be_nil.or be >= 114688
should eq('sha256').or eq('sha384').or eq('sha512')
should be_nil.or eq('True').or eq('auto')
should be <= MANAGEABLE_CONTAINER_NUMBER
should be_empty
should_not contain_duplicates
should include rule
should_not include('')
should_not include('.')
should match(/^root|syslog$/)
should match(/^INCREMENTAL|INCREMENTAL_ASYNC$/)
should match(/^\s*?SSLHonorCipherOrder\s+?On/i)
should match(/-FollowSymLinks/).or match(/\+SymLinksIfOwnerMatch/)
should cmp 'root'
should be < Date.today - 90 - Date.new(1970,1,1)
Attributes
An attribute is a parameter that InSpec reads from a YAML file provided on the command line. You can use this feature either to change a profile’s behavior by passing different attribute files or to store secrets that should not be directly present in a profile.

# attributes
CLIENT_MAX_BODY_SIZE = attribute(
  'client_max_body_size',
  description: 'Sets the maximum allowed size of the client request body, specified in the “Content-Length” request header field. If the size in a request exceeds the configured value, the 413 (Request Entity Too Large) error is returned to the client. Please be aware that browsers cannot correctly display this error. Setting size to 0 disables checking of client request body size.',
  default: '1k'
)

CLIENT_BODY_BUFFER_SIZE = attribute(
  'client_body_buffer_size',
  description: 'Sets buffer size for reading client request body. In case the request body is larger than the buffer, the whole body or only its part is written to a temporary file. By default, buffer size is equal to two memory pages. This is 8K on x86, other 32-bit platforms, and x86-64. It is usually 16K on other 64-bit platforms.',
  default: '1k'
)
Best Practices
Avoid native ruby IO functions
Avoid:

File.new(“filename”).read
File.read(“filename”)
IO.read(“filename”)
Use:

file(“filename”)
Edge Cases
Test not implemented
control 'foo-1' do
  describe 'bar-test' do
    skip 'Not implemented yet'
  end
end
only_if
# Avoid putting conditionals outside control blocks
if package('..').installed?
  control 'redis-1' do
    ...
  end
end

# Instead, always enclose it in a control (or better in a describe block):
control 'redis1' do
  only_if('redis is not installed.') do
    command('redis-cli').exist?
  end

  describe command('redis-cli SET test_inspec "HELLO"') do
    its('stdout') { should match /OK/ }
  end
end

# ----

# Format
only_if { <something>.exists? }
only_if { <something> <comparison operator> <state> }

# Examples
only_if { HTTP_METHODS_CHECK != false }
only_if { os.redhat? }
only_if { os.name != 'fedora' && !container_execution }

control "package-test1" do
  only_if { package('..').installed? }
end

only_if do
  # Abort if service not found
  command(apache.service).exist?
end

# Only run test if criteria match
control 'foo-1' do
  only_if { os[:family] != 'ubuntu' && os[:release] != '16.04' } || only_if { os[:family] != 'debian' && os[:release] != '8' }
  describe service(apache.service) do
    it { should be_enabled }
  end
end

control 'foo-2' do
  logging_conf = file("#{keystone_conf_dir}/logging.conf")

  if logging_conf.exist?
    describe logging_conf do
      its('mode') { should cmp '0640' }
    end
  end
end

# run test if file does not exist
control 'os-10' do
  impact 1.0
  title 'CIS: Disable unused filesystems'
  desc '1.1.1 Ensure mounting of cramfs, freevxfs, jffs2, hfs, hfsplus, squashfs, udf, FAT'
  only_if { !container_execution }
  efi_dir = inspec.file('/sys/firmware/efi')
  describe file('/etc/modprobe.d/dev-sec.conf') do
    its(:content) { should match 'install cramfs /bin/true' }
    its(:content) { should match 'install freevxfs /bin/true' }
    its(:content) { should match 'install jffs2 /bin/true' }
    its(:content) { should match 'install hfs /bin/true' }
    its(:content) { should match 'install hfsplus /bin/true' }
    its(:content) { should match 'install squashfs /bin/true' }
    its(:content) { should match 'install udf /bin/true' }
    # if efi is active, do not disable vfat. otherwise the system
    # won't boot anymore
    unless efi_dir.exist?
      its(:content) { should match 'install vfat /bin/true' }
    end
  end
end
Default value
# Set default value
shadow_group = 'root'
shadow_group = 'shadow' if os.debian? || os.suse?

# Single lines can be broken up with a trailing backslash
audit_pkg = os.redhat? \
         || os.suse? \
         || os.name == 'amazon' \
         || os.name == 'fedora' ? 'audit' : 'auditd'

# In a control
control 'foo-1' do
 if os.redhat? || os.name == 'fedora'
    describe file('/etc/shadow') do
      it { should_not be_writable.by('owner') }
      it { should_not be_readable.by('owner') }
    end
  else
    describe file('/etc/shadow') do
      it { should be_writable.by('owner') }
      it { should be_readable.by('owner') }
    end
  end
end
Hide sensitive output
describe file('/tmp/mysecretfile'), :sensitive do
  its('content') { should match /secret_info/ }
end
Pass if one test succeeds
With InSpec it is possible to check if at least one of a collection of checks is true. For example: If a setting is configured in two different locations, you may want to test if either configuration A or configuration B have been set. This is accomplished via describe.one. It defines a block of tests with at least one valid check.

describe.one do
  describe ConfigurationA do
    its('setting_1') { should eq true }
  end

  describe ConfigurationB do
    its('setting_2') { should eq true }
  end
end
Complex examples
Iterate over whitespace array
hab_env_vars = %w(
  HAB_AUTH_TOKEN
  HAB_CACHE_KEY_PATH
  HAB_DEPOT_URL
  HAB_ORG
  HAB_ORIGIN
  HAB_ORIGIN_KEYS
  HAB_RING
  HAB_RING_KEY
  HAB_STUDIOS_HOME
  HAB_STUDIO_ROOT
  HAB_USER
)

hab_env_vars.each do |e|
  describe os_env(e) do
    its('content') { should eq nil }
  end
end
Prepare loop and iterate over combined elements
module_path = File.join(apache.conf_dir, '/mods-enabled/')
loaded_modules = command('ls ' << module_path).stdout.split.keep_if { |file_name| /.load/.match(file_name) }

loaded_modules.each do |id|
  describe file(File.join(module_path, id)) do
    its('content') { should_not match(/^\s*?LoadModule\s+?dav_module/) }
    its('content') { should_not match(/^\s*?LoadModule\s+?cgid_module/) }
    its('content') { should_not match(/^\s*?LoadModule\s+?cgi_module/) }
    its('content') { should_not match(/^\s*?LoadModule\s+?include_module/) }
  end
end

# ----

# Use the InSpec resource to enumerate IDs, then test in-depth
docker.containers.running?.ids.each do |id|
  describe docker.object(id) do
    its('State.Health.Status') { should eq 'healthy' }
  end
end
Parse nginx config
# determine all required paths
nginx_path          = '/etc/nginx'
nginx_conf          = File.join(nginx_path, 'nginx.conf')
nginx_confd         = File.join(nginx_path, 'conf.d')
nginx_enabled       = File.join(nginx_path, 'sites-enabled')
nginx_parsed_config = command('nginx -T').stdout

options = {
  assignment_regex: /^\s*([^:]*?)\s*\ \s*(.*?)\s*;$/
}

options_add_header = {
  assignment_regex: /^\s*([^:]*?)\s*\ \s*(.*?)\s*;$/,
  multiple_values: true
}

control 'nginx-07' do
  describe user(nginx_lib.valid_users) do
    it { should exist }
  end

  describe parse_config_file(nginx_conf, options) do
    its('user') { should eq nginx_lib.valid_users }
  end

  describe parse_config_file(nginx_conf, options) do
    its('group') { should_not eq 'root' }
  end
end

control 'nginx-08' do
  impact 1.0
  title 'Prevent clickjacking'
  desc 'Do not allow the browser to render the page inside an frame or iframe.'
  describe parse_config(nginx_parsed_config, options_add_header) do
    its('add_header') { should include 'X-Frame-Options SAMEORIGIN' }
  end
end

control 'nginx-09' do
  impact 1.0
  title 'Enable Cross-site scripting filter'
  desc 'This header is used to configure the built in reflective XSS protection. This tells the browser to block the response if it detects an attack rather than sanitising the script.'
  describe parse_config(nginx_parsed_config, options_add_header) do
    its('add_header') { should include 'X-XSS-Protection "1; mode=block"' }
  end
end

control 'nginx-10' do
  impact 1.0
  title 'Disable content-type sniffing'
  desc 'It prevents browser from trying to mime-sniff the content-type of a response away from the one being declared by the server. It reduces exposure to drive-by downloads and the risks of user uploaded content that, with clever naming, could be treated as a different content-type, like an executable.'
  describe parse_config(nginx_parsed_config, options_add_header) do
    its('add_header') { should include 'X-Content-Type-Options nosniff' }
  end
end
Validate ini file
control 'check-identity-04' do
  title 'Strong hashing algorithms should be used for PKI tokens'
  ref 'http://docs.openstack.org/security-guide/identity/checklist.html#check-identity-04-does-identity-use-strong-hashing-algorithms-for-pki-tokens'

  keystone_conf = ini(keystone_conf_file)

  only_if do
    keystone_conf.value(['token', 'provider']) == 'pki'
  end

  describe ini(keystone_conf_file) do
    its(['token', 'hash_algorithm']) { should eq('sha256').or eq('sha384').or eq('sha512') }
  end
end

control 'foo-1' do
  describe ini(keystone_conf_file) do
    its(['oslo_middleware', 'max_request_body_size']) { should be_nil.or be >= 114688 }
  end

  describe ini("#{conf_dir}/ceilometer.conf") do
    its(['DEFAULT', 'auth_strategy']) { should be_nil.or eq 'keystone' }
    its(['keystone_authtoken', 'auth_uri']) { should match(/^https:/) }
  end

  describe ini(keystone_conf_file) do
    its(['DEFAULT', 'nas_secure_file_permissions']) { should be_nil.or eq('True').or eq('auto') }
  end
end
Filtering
Terminology
Filter statement
When using a plural resource, a filter statement is used to select individual test subjects using filter criteria. A filter statement almost always is indicated by the keyword where, e.g. .where(...) and may be repeated using method chaining, e.g. .where(...).where(...).

A filter statement may use method call syntax (which allows basic criteria operations, such as equality, regex matching, and ruby === comparison) or block syntax (which allows arbitrary code, e.g. .where {...}).

Filter criteria
When using a plural resource, a filter criterion is used to select individual test subjects within a filter statement. You may use multiple filter criteria in a single filter statement, .where(color: 'blue', size: 'small').

When method-call syntax is used with the filter statement, you provide filter criteria as a Hash, with filter criteria names as keys, and conditions as the Hash values. You may provide test, true/false, or numbers, in which case the comparison is equality; or you may provide a regular expression, in which case a match is performed.

Here, (color: blue) is a single filter criterion being used with a filter statement in method-call syntax.

# Count only blue cars
describe cars.where(color: 'blue') do
  its('count') { should eq 20 }
end
When block-method syntax is used with the filter statement, you provide a block. The block may contain arbitrary code, and each filter criteria will be available as an accessor. The block will be evaluated once per row, and each block that evaluates to a truthy value will pass the filter.

Here, { engine_cylinders >= 6 } is a block-syntax filter statement referring to one filter criterion.

describe cars.where { engine_cylinders >= 6 } do
  its('city_mpg_ratings') { should_not include '4-star' }
end
Examples
# Method form, simple
# Select just the root user (direct equality)
describe shadow.where(user: 'root') do
  its ('count') { should eq 1 }
end

# Method form, with a regex
# Select all users whose names begin with smb
describe shadow.where(user: /^smb/) do
  its ('count') { should eq 2 }
end

# Block form
# Select users whose passwords have expired
describe shadow.where { expiry_date > 0 } do
  # This test directly asserts that there should be 0 such users
  its('count') { should eq 0 }
  # But if the count test fails, this test outputs the users that are causing the failure.
  its('users') { should be_empty }
end

# ----

# For improved readability put longer filter criteria 
# on separate lines
describe resource(
  property1_name: 'my-filter',
  property2_name: 'my-log-group'
) do
  it { should exist }
end

# ----

# Ensure everyone except admins has an stale policy of no more than 14 days
describe shadow.where { user !~ /adm$/ } do
  its('inactive_days.max') { should be <= 14 }
end

# Find 'locked' accounts and ensure 'nobody' is on the list
describe shadow.where(password: '*LK*') do
  its('users') { should include 'nobody' }
end

# Find users who have not changed their password within 90 days
describe shadow.where { last_change > Date.today - 90 - Date.new(1970,1,1) } do
  its('users') { should be_empty }
end

# Find users who have a nonzero wait time
describe shadow.where { min_days > 0 } do
  its('users') { should be_empty }
end

# All users should have a 30-day policy
describe shadow.where { max_days != 30 } do
  its('users') { should be_empty }
end

# Ensure everyone has a stale policy of no more than 14 days.
describe shadow.where { inactive_days.nil? || inactive_days > 14 } do
  its('users') { should be_empty }
end

# Ensure no one is disabled due to a old password
describe shadow.where { !expiry_date.nil? } do
  its('users') { should be_empty }
end

# Ensure no one is disabled for more than 14 days
describe shadow.where { !expiry_date.nil? && expiry_date - Date.new(1970,1,1) > 14} do
  its('users') { should be_empty }
end

describe auditd.syscall('open') do
# or
describe auditd.where(syscall: 'open') do

describe auditd.syscall('chown').where { arch == "b32" } do
# or
describe auditd.where(syscall: 'chown', arch: 'b32' ) do
Common ruby methods
See Ruby Docs

cmd.stdout.chomp
Will remove trailing new lines from the string, i.e. it removes carriage return characters (\n, \r, and \r\n).

cmd.exit_status.to_i.nonzero?
Test if a command was unsuccessful.

array.keep_if
Deletes every element of self for which the given block evaluates to false.

a = %w{ a b c d e f }
a.keep_if { |v| v =~ /[aeiou]/ }  # => ["a", "e"]
array.sort
No explanation necessary.

array.uniq
Removes duplicate elements from array. Returns nil if no changes are made (that is, no duplicates are found).

a = [ "a", "a", "b", "b", "c" ]
a.uniq   # => ["a", "b", "c"]

b = [ "a", "b", "c" ]
b.uniq   # => nil
array.flatten
Recursively turns nested arrays into a one-dimensional flattening of self.

a = [[1, 2, 3], [4, 5, 6, [7, 8]], 9, 10]
a.flatten  # => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
string.gsub
The gsub string method replaces a substring with another string. It finds all instances of the matched string and replaces it with the new argument. The method takes two arguments. The first is the text you want to replace and the second is the new text.

sentence = "Today is Friday"
sentence.gsub('Friday','Saturday')  # => "Today is Saturday"

sentence = "My favourite number is 3"
sentence.gsub(/\d+/,"7") # => "My favourite number is 7"
An entire string, as opposed to a substring, may be replaced using the replace method:

sentence = "Today is Friday" 
sentence.replace("Tomorrow is Saturday") # => "Tomorrow is Saturday"
string.scan
Iterate through str, matching the pattern (which may be a Regexp or a String). For each match, a result is generated and either added to the result array. If the pattern contains no groups, each individual result consists of the matched string, $&. If the pattern contains groups, each individual result is itself an array containing one entry per group.

a = "cruel world"
a.scan(/\w+/)        #=> ["cruel", "world"]
a.scan(/.../)        #=> ["cru", "el ", "wor"]
a.scan(/(...)/)      #=> [["cru"], ["el "], ["wor"]]
a.scan(/(..)(..)/)   #=> [["cr", "ue"], ["l ", "wo"]]
string.split
Divides str into substrings based on a delimiter, returning an array of these substrings.

If pattern is a String, then its contents are used as the delimiter when splitting str.
If pattern is a single space, str is split on whitespace, with leading whitespace and runs of contiguous whitespace characters ignored.
If pattern is a Regexp, str is divided where the pattern matches. Whenever the pattern matches a zero-length string, str is split into individual characters.
If pattern contains groups, the respective matches will be returned in the array as well.
If pattern is nil, the value of $; is used. If $; is nil (which is the default), str is split on whitespace as if ‘ ‘ were specified.
If the limit parameter is omitted, trailing null fields are suppressed. If limit is a positive number, at most that number of split substrings will be returned (captured groups will be returned as well, but are not counted towards the limit). If limit is 1, the entire string is returned as the only entry in an array. If negative, there is no limit to the number of fields returned, and trailing null fields are not suppressed.
When the input str is empty an empty Array is returned as the string is considered to have no fields to split.
" now's  the time".split        #=> ["now's", "the", "time"]
" now's  the time".split(' ')   #=> ["now's", "the", "time"]
" now's  the time".split(/ /)   #=> ["", "now's", "", "the", "time"]
"1, 2.34,56, 7".split(%r{,\s*}) #=> ["1", "2.34", "56", "7"]
"hello".split(//)               #=> ["h", "e", "l", "l", "o"]
"hello".split(//, 3)            #=> ["h", "e", "llo"]
"hi mom".split(%r{\s*})         #=> ["h", "i", "m", "o", "m"]

"mellow yellow".split("ello")   #=> ["m", "w y", "w"]
"1,2,,3,4,,".split(',')         #=> ["1", "2", "", "3", "4"]
"1,2,,3,4,,".split(',', 4)      #=> ["1", "2", "", "3,4,,"]
"1,2,,3,4,,".split(',', -4)     #=> ["1", "2", "", "3", "4", "", ""]

"1:2:3".split(/(:)()()/, 2)     #=> ["1", ":", "", "", "2:3"]

"".split(',', -1)               #=> []
string.tr
tr(from_str, to_str) : Returns a copy of str with the characters in from_str replaced by the corresponding characters in to_str. If to_str is shorter than from_str, it is padded with its last character in order to maintain the correspondence.

"hello".tr('el', 'ip')      #=> "hippo"
"hello".tr('aeiou', '*')    #=> "h*ll*"
"hello".tr('aeiou', 'AA*')  #=> "hAll*"
Both strings may use the c1-c2 notation to denote ranges of characters, and from_strmay start with a ^, which denotes all characters except those listed.

"hello".tr('a-y', 'b-z')    #=> "ifmmp"
"hello".tr('^aeiou', '*')   #=> "*e**o"
The backslash character </code> can be used to escape <code>^ or - and is otherwise ignored unless it appears at the end of a range or the end of the from_str orto_str:

"hello^world".tr("\\^aeiou", "*") #=> "h*ll**w*rld"
"hello-world".tr("a\\-eo", "*")   #=> "h*ll**w*rld"

"hello\r\nworld".tr("\r", "")   #=> "hello\nworld"
"hello\r\nworld".tr("\\r", "")  #=> "hello\r\nwold"
"hello\r\nworld".tr("\\\r", "") #=> "hello\nworld"

"X['\\b']".tr("X\\", "")   #=> "['b']"
"X['\\b']".tr("X-\\]", "") #=> "'b'"
.to_str / .to_i
No explanation necessary.

Examples
<some_resource>.forwarding_rules.flatten.each { |rule|
    describe rule do
        its('property1') { should eq "https" }
        its('property2') { should eq 443 }
    end
}

# ----

# 1. build path string
# 2. use file resource on string
# 3. access the file resource's "content" property
# 4. remove commented lines
# 5. detect all virtualhost stanzas (case [i]nsensitive, [m]ultiline)
# 6. store nested virtualhosts as new elements in a one-dimensional array

# Regex are created with `/pat/` and `%r{pat}`

# https://github.com/dev-sec/apache-baseline/blob/master/controls/apache_spec.rb
virtual_host = file(
    File.join(sites_enabled_path, id)
  )
  .content.gsub(/#.*$/, '')
  .scan(%r{<virtualhost.*443(.*?)<\/virtualhost>}im).flatten
  
next if virtual_host.empty?
Profiles
Profile Structure
A profile should have the following structure:

examples/profile
├── README.md
├── controls
│   ├── example.rb
│   └── control_etc.rb
├── libraries
│   └── extension.rb
|── files
│   └── extras.conf
└── inspec.yml
where:

inspec.yml includes the profile description (required)
controls is the directory in which all tests are located (required)
libraries is the directory in which all InSpec resource extensions are located (optional)
files is the directory with additional files that a profile can access (optional)
README.md should be used to explain the profile, its scope, and usage
Create a Profile
InSpec version: 3.2.6:

$ inspec init help profile
Usage:
  inspec init profile [OPTIONS] NAME

Options:
  p, [--platform=PLATFORM]             # Which platform to generate a platform for: choose from os, aws, gcp
                                       # Default: os
      [--overwrite], [--no-overwrite]  # Overwrites existing directory
      [--log-level=LOG_LEVEL]          # Set the log level: info (default), debug, warn, error
      [--log-location=LOG_LOCATION]    # Location to send diagnostic log messages to. (default: STDOUT or Inspec::Log.error)

Generate a new profile
InSpec Shell
List resources and their respective help:

$ inspec shell

help resources
help <resource name>
Libraries
mongo example
Source

resource_mongo_command.rb
nginx example
Source

nginx_lib.rb
Misc. (unsorted/wip)
  describe file('/proc/sys/kernel/random/entropy_avail').content.to_i do
    it { should >= 1000 }
  end

  describe file(json('/etc/docker/daemon.json').params['tlscacert']) do
    it { should exist }
    it { should be_file }
    it { should be_owned_by 'root' }
    it { should be_grouped_into 'root' }
  end
control 'foo-1' do
  only_if { os.linux? }

  # use custom library to test if a systemd unit for a socket config
  # exists and return a message if not
  if docker_helper.socket
    rule = '-w ' + docker_helper.socket + ' -p rwxa -k docker'
    describe auditd do
      its(:lines) { should include(rule) }
    end
  else
    describe 'audit docker service' do
      skip 'Cannot determine docker socket'
    end
  end
end
control 'foo-1' do
  instantiated_images = command('docker ps -qa | xargs docker inspect -f \'.Image\'').stdout.split
  all_images = command('docker images -q --no-trunc').stdout.split
  diff = all_images - instantiated_images

  describe diff do
    it { should be_empty }
  end
end

# ----

control 'foo-2' do
  total_on_host = command('docker info').stdout.split[1].to_i
  total_running = command('docker ps -q').stdout.split.length
  diff = total_on_host - total_running

  describe diff do
    it { should be <= MANAGEABLE_CONTAINER_NUMBER }
  end
end

control 'foo-3' do
  total_tasks = command("ps aux | grep #{apache.service} | grep -v grep | grep root | wc -l | tr -d [:space:]").stdout.to_i

  describe total_tasks do
    it { should eq 1 }
  end
end
[5000, 35357].each do |port|
  # TODO: workaround until https://github.com/chef/inspec/issues/1205 is fixed
  next if os.name.nil? # detect mock backend during inspec check
  describe ssl(port: port) do
    it { should be_enabled }
  end
end
Pass options to resource
options = {
  enable_remote_worker: true,
  headers: { 'User-Agent' => 'Docker' },
  ssl_verify: false
}

describe http('https://localhost', options) do
  its('status') { should cmp 200 }
end