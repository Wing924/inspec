# VulcanoSec Specs

VulcanoSec specs is a collection of resources and matchers to test the compliance of your nodes. This documentation provides an introduction to this mechanism and shows how to write custom tests.

### Introduction

At first, we add our tests to the `test` folder. Each test file must end with `_spec.rb`:

    mkdir test
    touch test/example_spec.rb

We add a rule to this file, to check the `/tmp` path in our system:

```ruby
# encoding: utf-8

rule "cis-fs-2.1" do                        # A unique ID for this rule
  impact 0.7                                # The criticality, if this rule fails.
  title "Create separate /tmp partition"    # A human-readable title
  desc "An optional description..."
  describe file('/tmp') do                  # The actual test
    it { should be_mounted }
  end
end
```

Let's add another spec for checking the SSH server configuration:

    touch test/sshd_spec.rb

It will contain:

```ruby
# encoding: utf-8

# Skip all rules, if SSH doesn't exist on the system
only_if do
  command('sshd').exists?
end

rule "sshd-11" do
  impact 1.0
  title "Server: Set protocol version to SSHv2"
  desc "
    Set the SSH protocol version to 2. Don't use legacy
    insecure SSHv1 connections anymore.
  "
  describe sshd_config do
    its('Protocol') { should eq('2') }
  end
end

rule "sshd-7" do
  impact 1.0
  title "Server: Do not permit root-based login with password."
  desc "
    To reduce the potential to gain full privileges
    of a system in the course of an attack (by either misconfiguration
    or vulnerabilities), do not allow login as root with password
  "
  describe sshd_config do
    its('PermitRootLogin') { should match(/no|without-password/) }
  end
end
```