he Vagrant sync folder is /home/vagrant/sync instead of /vagrant (which is the Vagrant default). This will be changed in the next release.
The root password is set to a random string, instead of "vagrant". Use sudo as the vagrant user to gain administrative privileges, no password is required.
The VirtualBox Guest Additions are not preinstalled, and there are currently no plans of adding them. They are only needed for shared folders; host-only networking and forwarded ports work, although Vagrant displays a warning to the contrary. If you use Ansible, take a look at https://github.com/lpancescu/cloud-instance-starter-kit for an example of automatic installation. The vagrant-vbguest plugin might also work (not tested).




curl -s https://atlas.hashicorp.com/api/v1/box/puppetlabs/centos-5.11-64-nocm/version/1.0.2 | python -m json.tool
{
    "created_at": "2015-07-29T15:32:18.312Z",
    "description_html": "<ul>\n<li>version 1.0.2, see the <a href=\"https://github.com/puppetlabs/puppetlabs-packer/tree/master/templates/centos-5.11/CHANGELOG\">changelog</a> for details</li>\n</ul>\n",
    "description_markdown": "\n* version 1.0.2, see the [changelog](https://github.com/puppetlabs/puppetlabs-packer/tree/master/templates/centos-5.11/CHANGELOG) for details\n ",
    "number": "1.0.2",
    "providers": [
        {
            "created_at": "2015-07-29T15:32:19.356Z",
            "download_url": "https://atlas.hashicorp.com/puppetlabs/boxes/centos-5.11-64-nocm/versions/1.0.2/providers/vmware_fusion.box",
            "hosted": false,
            "hosted_token": null,
            "name": "vmware_fusion",
            "original_url": "https://s3.amazonaws.com/puppetlabs-vagrantcloud/centos-5.11-x86_64-vmware-nocm-1.0.2.box",
            "updated_at": "2015-08-12T04:45:51.188Z"
        },
        {
            "created_at": "2015-07-29T15:32:18.971Z",
            "download_url": "https://atlas.hashicorp.com/puppetlabs/boxes/centos-5.11-64-nocm/versions/1.0.2/providers/vmware_desktop.box",
            "hosted": false,
            "hosted_token": null,
            "name": "vmware_desktop",
            "original_url": "https://s3.amazonaws.com/puppetlabs-vagrantcloud/centos-5.11-x86_64-vmware-nocm-1.0.2.box",
            "updated_at": "2015-08-12T04:45:51.219Z"
        },
        {
            "created_at": "2015-07-29T15:32:18.616Z",
            "download_url": "https://atlas.hashicorp.com/puppetlabs/boxes/centos-5.11-64-nocm/versions/1.0.2/providers/virtualbox.box",
            "hosted": false,
            "hosted_token": null,
            "name": "virtualbox",
            "original_url": "https://s3.amazonaws.com/puppetlabs-vagrantcloud/centos-5.11-x86_64-virtualbox-nocm-1.0.2.box",
            "updated_at": "2015-08-12T04:45:51.184Z"
        }
    ],
    "release_url": "https://atlas.hashicorp.com/api/v1/box/puppetlabs/centos-5.11-64-nocm/version/1.0.2/release",
    "revoke_url": "https://atlas.hashicorp.com/api/v1/box/puppetlabs/centos-5.11-64-nocm/version/1.0.2/revoke",
    "status": "active",
    "updated_at": "2015-08-12T04:45:51.227Z",
    "version": "1.0.2"
}

上面是vagrant手动下载的方式，我暂时是不考虑了，目前只考虑直接看原理了。

