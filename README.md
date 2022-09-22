# Ansible Execution Enviroment with Azure Collection

Credit: Original HowTo written by Peter Gustafsson / Richard Henshall.

The certified Ansible Azure Collection (https://github.com/ansible-collections/azure) has quite a few Python module dependencies, but that is easy to solve in Ansible Automation Platform 2.x with an Execution Environment (aka container template) that include these.

Here's a short HowTo on how to build this with ansible-builder and uploading it to Ansible Private Automation Hub on a Linux machine.

Install python, podman and wget:
```
$ sudo yum install python39 podman wget
```
Create a new clean python environment:
```
$ mkdir ~/ansible-builder && cd ~/ansible-builder
$ python3 -m venv builder
$ source builder/bin/activate
```
Install ansible / ansible-builder: (for use of Red Hat supported, refer to Ansible Automation Platform documentation)
```
$ pip install ansible
$ pip install ansible-builder
$ ansible --version
ansible [core 2.13.3]
```
For required build files, clone this repository and needed files to root in build environment.

### Needed files:

requirements.yml - This is where the collection(s) is defined.

requirements.txt - Python modules required by collection

(source: https://github.com/ansible-collections/azure/blob/dev/requirements-azure.txt)

bindep.txt - This files defines extra rpm's if needed

ee-ansible-azure.yml - The actual build file, more info bellow.

### Now, let's start building. First logon to registry.redhat.io with podman:
```
$ podman login registry.redhat.io
```
Let's check what ee images is available:
```
$ podman search registry.redhat.io/ee-minimal
```
```
registry.redhat.io/ansible-automation-platform/ee-minimal-rhel8...
registry.redhat.io/ansible-automation-platform-22/ee-minimal-rhel8...
registry.redhat.io/ansible-automation-platform-21/ee-minimal-rhel8...
registry.redhat.io/ansible-automation-platform-20-early-access/ee-minimal-rhel8...

```

For this project, we will use the minimal image. We will create a YAML-file to define this image and other dependencies. This file is called "ee-ansible.azure.yml" in this repo.

To build this for push to my Ansible Private Automation HUB, I'll also tag it with correct name like this:
```
$ ansible-builder build --tag <myAHurl>/ee-ansible-azure -f ee-ansible-azure.yml  -v 3
```
To list images:
```
$ podman images
```
Your freshly build ee-ansible.azure image should be visible in the list of images.

Last stretch is to upload the image to Ansible Private Automation HUB, or the container registry you prefer.

Given that you tagged the image correctly when building, you should be able to just do:
```
$ podman login <myAHurl>
$ podman push <myAHurl>/ee-ansible-azure
```
If you run a demo env without valid certificate, add "--tls-verify=false" to the podman statements.



