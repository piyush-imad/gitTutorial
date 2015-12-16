---
author: slowe
comments: true
date: 2015-12-16
layout: post
title: Modifying OpenStack Security Groups with Terraform
categories: Explanation
tags:
- Automation
- OpenStack
- CLI
- OSS
---

In this post I'd like to discuss a potential (minor) issue with modifying [OpenStack][link-2] security groups with [Terraform][link-1]. I call this a "potential minor" issue because there is an easy workaround, which I'll detail in this post. I wanted to bring it to my readers' attention, though, because as of this blog post this matter had not yet been documented.

As you probably already know if you read [my recent introduction to Terraform blog post][xref-1], Terraform is a way to create _configurations_ that automate the creation or configuration of infrastructure components, possibly across a number of different providers and/or platforms. In the introductory blog post, I showed you how to write a Terraform configuration that would create an OpenStack logical network and subnet, create a logical router and attach it to the logical network, and then create an OpenStack instance and associate a floating IP. In that example, I used a key part of Terraform, known as _interpolation_.

Broadly speaking, interpolation allows Terraform to reference variables or attributes of other objects created by Terraform. For example, how does one refer to a network that he or she has just created? Here's an example taken from the introductory blog post:

{% highlight json %}
   "resource": {
        "openstack_compute_instance_v2": {
            "tf-instance": {
                "name": "tf-instance",
                "image_name": "${var.image}",
                "flavor_name": "${var.flavor}",
                "key_pair": "${var.key_pair}",
                "floating_ip": "${openstack_compute_floatingip_v2.tf-fip.address}",
                "security_groups": [
                    "default",
                    "basic-services"
                ],
                "network": {
                    "uuid": "${openstack_networking_network_v2.tf-net.id}"
                }
            }
        }
    }
{% endhighlight %}

I'll call your attention to the `floating_ip` and `uuid` lines, which reference an attribute from another object. In this case, I'm referencing the address of the floating IP object and the UUID of the network object.

Terraform can, as part of a configuration, create a security group. A simple security group might look something like this:

{% highlight json %}
    "resource": {
        "openstack_compute_secgroup_v2": {
            "ssh": {
                "name": "ssh",
                "description": "Allow SSH traffic",
                "rule": {
                    "from_port": "22",
                    "to_port": "22",
                    "ip_protocol": "tcp",
                    "cidr": "0.0.0.0/0"
                }
            }
        }
    }
{% endhighlight %}

As you can probably determine, this allows TCP port 22 (SSH) from any remote source. If you later wanted to associate this security group, you'd need to use interpolation to reference the security group when defining the instance.

Here's where the potential issue comes into play. You can use interpolation to reference the security group by its UUID, like this:

    "${openstack_compute_secgroup_v2.ssh.id}"

In this case, `ssh` is the name of the object (_not_ the value of the `name` attribute, although I'd recommend making them one and the same for your sanity), and you're referencing the UUID of the security group. This will work just fine..._until you need to modify the security group._

If you modify the security group in the Terraform configuration (such as adding a rule, removing a rule, or changing the values in a rule) and you're referencing (via interpolation) the security group by UUID, when you run `terraform plan` or `terraform apply` Terraform will become hopelessly confused and become unable to reconcile the state of your environment with the configuration. Running `terraform refresh` will _not_ recover it. You will have lost the ability to use Terraform to manage this particular set of infrastructure, period. (There may be a way to recover; I was unable to find any way to make it work again.)

Fortunately, there is a workaround, and it's a simple (albeit currently undocumented) one. Instead of referencing the security group's UUID via interpolation, reference the security group's name attribute, like this:

    "${openstack_compute_secgroup_v2.ssh.name}"

(Again, I'll remind you that `ssh` in the above example is the name of the object, _not_ the value of the `name` attribute.) By simply switching from referencing the UUID to referencing the name, you'll work around the issue with updating security groups, and you'll once again be able to freely modify the security group's configuration using Terraform.

As you can see in [the GitHub issue for this problem][link-3], it's likely that this potential problem will be documented in the Terraform documentation soon.


[link-1]: http://terraform.io
[link-2]: http://www.openstack.org
[link-3]: https://github.com/hashicorp/terraform/issues/4319
[xref-1]: {% post_url 2015-11-25-intro-to-terraform %}