---
layout: post
status: publish
published: true
title: Monster VMs Mass Storage & How to check storage used
author:
  display_name: iguy
  login: iguy
  email: iguy@ionsphere.org
  url: ''
author_login: iguy
author_email: iguy@ionsphere.org
wordpress_id: 712
wordpress_url: http://itsjustanotherlayer.com/?p=712
date: 2012-11-07
categories:
- VMware
tags:
- esxi
- PowerCLI
- scripts
- monsterVM
comments: []
---
Jason Boche posted an [ESXi Heap Size](http://www.boche.net/blog/index.php/2012/09/12/monster-vms-esxi-heap-size-trouble-in-storage-paradise/) interesting discovery with regards to Monster VMs and the amount of storage one would be sticking to them.  One of the large projects being designed today  has the likely reason to hit this limit with just a single VM.  This is pretty easy for that project to hit that as a single VM will have about 75TB assigned to it.   Since this design has 4 or 5 VMs of this size, which then means our standard ESXi Host will have around 300+ TB assigned to it.  Thankfully RDM isn't impacted by this limit in theory, so there is a workaround available.  Not ideal and yet doable.

Instead one of the questions that came out of this news is "Does this issue impact the rest of our environment?"   Will this explain some random stability issues we have seen?

This script will return what every single host has in terms of VMDK files.  To use this script look at each host and see if the host itself is under the limits.  Then look at the cluster level based on the HA rules and see what the VMDK limits would be if a host failed.  That should give an idea of how close your clusters might be to hitting this limit.

{% highlight powershell linenos %}
if ( -not (Get-PSSnapIn | where {$_.Name -eq "VMware.VimAutomation.Core"}) )
{
    Add-PSSnapin -Name "VMware.VimAutomation.Core"
}
$vcenters = "vCenter"
connect-viserver -server $vcenters -cred (Get-Credential)
$diskuse = @()
$clusters = get-cluster
foreach ($cluster in $clusters) {
    "Working on Cluster $cluster"
    $vmhosts = get-vmhost -Location $cluster
    foreach ($vmhost in $vmhosts) {
    "`tWorking on VMHost $vmhost"
        $vmlist = get-vm -location $vmhost
        $totalDisk = 0
        foreach ($vm in $vmlist) {
            $view = $vm | get-View
            foreach ($disk in $view.Guest.Disk) {
                $disksize = ([math]::Round($disk.Capacity/1MB))
                $totaldisk += $disksize
            }
         }
         "`t`tVM size: $totaldisk"
            $obj = new-object PSObject -Property @{
                Cluster = $cluster
                VMHost  = $vmhost
                TotalDiskMB = $totaldisk
            }
         $diskuse += $obj
    }
}
$diskuse | export-csv "OpenFileDiskUse.csv" -NoTypeInformation
{% endhighlight %}

With Default settings the magic numbers to look for:
<ul>
<li>ESXi 4.x - 4TB</li>
<li>ESXi 5.1 - 8TB</li><br />
</ul></p>
