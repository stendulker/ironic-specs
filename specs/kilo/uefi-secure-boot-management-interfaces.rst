..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================================================
Ironic Management Interfaces to support UEFI Secure Boot
========================================================

https://blueprints.launchpad.net/ironic/+spec/uefi-secure-boot-management-
interfaces

Some of the Ironic deploy drivers support UEFI boot. It would be useful to
security sensitive users to deploy more securely using Secure Boot feature
of the UEFI. This spec proposes Ironic Management APIs that would help in UEFI
Secure Boot deployment of Baremetal.

Problem description
===================

Secure Boot is part of the UEFI specification (http://www.uefi.org). It helps
to make sure that node boots using only software that is trusted by Admin/End
user.

Secure Boot is different from TPM (Trusted Platform Module). TPM is a standard
for a secure cryptoprocessor, which is dedicated microprocessor designed to
secure hardware by integrating cryptographic keys into devices. Secure Boot is
part of UEFI specification, which can secure the boot process by preventing
the loading of drivers or OS loaders that are not signed with an acceptable
digital signature.

When the node starts with secure boot enabled, system firmware checks the
signature of each piece of boot software, including firmware drivers (Option
ROMs), boot loaders and the operating system. If the signatures are good,
the node boots, and the firmware gives control to the operating system.

The Admin and End users having security sensitivity with respect to baremetal
provisioning owing to the workloads they intend to run on the provisioned
nodes would be interested in using secure boot provided by UEFI.

Once secure boot is enabled for a node, it cannot boot using unsigned boot
images. Hence it is important to use signed bootloaders and kernel during
deploy and regular boot of the node if node were to be booted using secure
boot.

UEFI Secure Boot feature is driven by UEFI standards and hence most of the
vendor implementation are similar. It would be useful to have standardized
Ironic Management APIs to support the same. Currently there are no Ironic
Management interfaces related to UEFI secure boot.

Proposed change
===============

It would be useful to have following Management interfaces to support UEFI
secure boot.

def set_secure_boot_state(self, task, flag):

""" Enable or disable UEFI Secure Boot for the next boot
  :param task: a task from TaskManager.
  :param flag: Boolean value to indicate enable or disable of UEFI secure boot

"""

def get_secure_boot_state(self, task):

""" Retrieves current enabled state of UEFI secure boot
  :param task: a task from TaskManager.
  :return: Boolean value indicating current state of secure boot.

"""

There two more operations that are possible in secure boot environment if user
were to use the custom signatures, viz:

* Clear all the keys related to secure boot in next boot

* Reset to default secure boot keys on next boot.

But these operations to be done during teardown and can be performed as part
of zapping stage and can be added as the zap steps.

Alternatives
------------

Use vendor-passthrough API for the proposed Management interfaces.

Data model impact
-----------------

The proposed Management APIs are only for the driver implementation. These
APIs would be called by drivers during deploy to enable or disable secure boot
based on the deploy request. There is no need for the REST API endpoint for
the same as they are to be used only during deploy.
The secure boot enable/disable operation would get carried out as part of Nova
boot flavor. User would not require to call these APIs directly.

REST API impact
---------------

None

RPC API impact
--------------

No new RPC API would be added for any of the proposed two Management
interfaces.

Driver API impact
-----------------

The new proposed Management interfaces need not implemented by all the
drivers. The base.ManagementInterface implementation would throw
'UnsupportedDriverExtension' for the same.

Nova driver impact
------------------

None

Security impact
---------------

In the absence of local agent on the deployed node, Ironic does not have any
mechanism to know if the deployed node has booted up successfully. This is a
known limitation and is applicable to secure boot images being deployed.

Other end user impact
---------------------

The images to be used in secure boot deploy needs to be signed and these
signatures should be available at the node in UEFI NVRAM for the firmware to
validate during boot. The OEM systems are preloaded with signatures of some
of the OS vendors. For the OS images of such vendor, deployer can directly
deploy such images in secure boot environment. If the deployer uses any images
that are custom signed then before deploying such images, deployer needs to
manually enroll those certificates into the node's UEFI NVRAM. It is also
important that deployer has flashed the firmware which is digitally signed and
its signature is part of the UEFI NVRAM.

Scalability impact
------------------

None

Performance Impact
------------------

None

Other deployer impact
---------------------

The images to be used in secure boot deploy needs to be signed and these
signatures should be available at the node in UEFI NVRAM for the firmware to
validate during boot. The OEM systems are preloaded with signatures of some of
the OS vendors. For the OS images of such vendors, deployer can directly
deploy such images in secure boot environment. If the deployer uses any images
that are custom signed then before deploying such images, deployer needs to
manually enroll those certificates into the node's UEFI NVRAM. It is also
important that deployer has flashed the firmware which is digitally signed and
its signature is part of the UEFI NVRAM.

Developer impact
----------------

Proposing as a standard interface , vendor-specific drivers can use the same
if required.

Implementation
==============

Assignee(s)
-----------

primary author and contact.

Primary assignee:
  Shivanand Tendulker (stendulker@gmail.com)

Work Items
----------

1. Implement changes for new Management interfaces in REST and conductor
   using RIS for iLO driver.

Dependencies
============

None

Testing
=======

Unit tests would be added for all newly added code.

Upgrades and Backwards Compatibility
====================================

None

Documentation Impact
====================

Newly added functionality would be appropriately documented.

References
==========

None

