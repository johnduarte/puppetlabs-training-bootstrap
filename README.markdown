# Bootstrap CentOS VMs for training
Installs the training, student, or learning environments on an existing VM.

## Usage
To turn the current machine or VM into one of the Education environments.
NOTE: This changes the hostname and should probably only be done from within a Centos 6.5 or 6.6 32bit base VM.  The old rakefile has been deprecated and can be found in Rakefile.orig

The basic process is to start a new VM, check out this repo within it, and run `rake VMNAME` from the root of the repo.

e.g. for a training VM for classroom use:
- Build a new VM and ssh to it
- `git clone https://github.com/puppetlabs/puppetlabs-training-bootstrap bootstrap`
- `cd bootstrap`
- `rake training`

## Packer
Packer scripts are provided in the `packer` directory. These depend on vmware fusion and the ovftool post-processor plugin from here: https://github.com/iancmcc/packer-post-processor-ovftool

Installing the post-processors can be a little tricky, if there are errors, go through all of the repos in gocode/src and do a `git pull` to make sure they're all up to date:

    for d in ~/gocode/src/*/*/*; do cd $d; git pull;done

The common configuration options have been set up in educationbase.json and vm specific variables are set in VMNAME.json
After the base VM is provisioned according to the settings in VMNAME.json, the bootstrap can be applied using educationbuild.json.

First create a base VM without any bootstrap applied:
- `packer build -var-file=student.json educationbase.json`

To initiate a packer build of the student vm on the base vm:
- `packer build -var-file=student.json educationbuild.json`


For the training vm follow the same two steps but with training.json:
- `packer build -var-file=training.json educationbase.json`
- `packer build -var-file=training.json educationbuild.json`

## Vagrant
There is a Vagrantfile that automates this process and builds on the puppetlabs/centos-6.6-32-nocm base box.
There are three boxes specified.

To start a student vagrant box:
- `vagrant up`

To start a training vagrant box for instructor use:
- `vagrant up training`

To start a learning vagrant box:
- `vagrant up learning`

## Updating PE version
The version of PE used to build the VM is determined by the pltraining-bootstrap module.
To update the version, set the value of `$pe_version` in `bootstrap::params`.

## Internal-only pre-release PE version builds
For pre-release builds:

1. Make a branch of this repo and of pltraining-bootstrap.
1. Edit the Puppetfile on this branch to point to your own fork and branch of pltraining-bootstrap.
1. Edit the build script in `/packer/scripts` to reference the correct branch of this repo at build time.
  * Note this change does not need to be commited to the branch.
1. Download the PE master and agent installers and place them in a `file_cache` directory in the root of this repository.
1. Update the `$pe_version` in `bootstrap::params` in your branch of pltraining-bootstrap.
1. Make sure to add, commit, and push those changes.
1. The packer build should run as ususal.
