Function Add-ESXiToCluster {
    <#
    .SYNOPSIS
        Add ESXi host to Cluster
    .PARAMETER ESXiName
        Specifies the name (DNS or IP) of the ESXi host that you want to add to the cluster
    .PARAMETER ESXiUser
        Specifies the user name you want to use for authenticating with the ESXi.
    .PARAMETER ESXiPassword
        Specifies the password you want to use for authenticating with the ESXi.
    .PARAMETER ClusterName
        Specifies the name of the cluster that you want to add a ESXi
    .EXAMPLE
        Add-ESXiToCluster -ESXiName '10.20.30.40' -ESXiUser 'root' -ESXiPassword 'thePassword' -ClusterName 'theCluster'
    .NOTES
        Copyright VMware, Inc. 2013.  All Rights Reserved.
    #>

    Param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$ESXiName,

        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$ESXiUser,

        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$ESXiPassword,

        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$ClusterName
    )

    "Retrieve Cluster: $ClusterName"
    $Cluster = Get-Cluster -Name $ClusterName

    "Add ESXi $ESXiName to Cluster $Cluster.Name"
    Add-VMHost -Name $ESXiName -User $ESXiUser -Password $ESXiPassword -Location $Cluster.Name -Force
}

Function New-VMwareCluster {
    <#
    .SYNOPSIS
        Create a new cluster on the location specified for the Location parameter
    .PARAMETER Location
        Specifies the location where you want to place the new cluster, if a datacenter is specified for the Location parameter, the cluster is created in its "hostFolder" folder.
    .PARAMETER ClusterName
        Specifies the name of the cluster that you want to create
    .EXAMPLE
        New-VMwareCluster -Location 'theDatacenter' -ClusterName 'theNewCluster'
    .NOTES
        Copyright VMware, Inc. 2010-2013.  All Rights Reserved.
    #>

    Param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$Location,

        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$ClusterName
    )

    "Create cluster $ClusterName at location $Location"
    New-Cluster -Location $Location -Name $ClusterName -DRSEnabled
}

Function Remove-VMwareCluster {
    <#
    .SYNOPSIS
        Deletes the specified clusters
    .PARAMETER ClusterName
        Specifies the clusters you want to remove
    .EXAMPLE
        Remove-VMwareCluster -ClusterName 'theCluster'
    .NOTES
        Copyright VMware, Inc. 2010-2013.  All Rights Reserved.
    #>

    Param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$ClusterName
    )

    "Retrieve Cluster $ClusterName"
    $Cluster =  Get-Cluster -Name $ClusterName

    "Remove Cluster $Cluster.name"
    Remove-Cluster $Cluster -Confirm:$false
}

Function New-VMwareDatacenter {
    <#
    .SYNOPSIS
        Creates a new datacenter in the location that is specified by the Location parameter
    .PARAMETER FolderName
        Specifies the location where you want to create the new datacenter
    .PARAMETER DatacenterName
        Specifies a name for the new datacenter
    .EXAMPLE
        New-VMwareDatacenter -FolderName 'theFolderName' -DatacenterName 'theDatacenter'
    .NOTES
        Copyright VMware, Inc. 2010-2013.  All Rights Reserved.
    #>

    Param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$FolderName,

        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$DatacenterName
    )

    New-Datacenter -Location $FolderName -Name $DatacenterName
}

Function Remove-VMwareDatacenter {
    <#
    .SYNOPSIS
        Removes the specified datacenter and their children objects
    .PARAMETER Datacenter
        Specifies the datacenters you want to remove
    .EXAMPLE
        Remove-VMwareDatacenter -DatacenterName fifthDatacenter
    .NOTES
        Copyright VMware, Inc. 2010-2013.  All Rights Reserved.
    #>

    Param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$DatacenterName
    )

    "Retrieve Datacenter $DatacenterName"
    $Datacenter =  Get-Datacenter -Name $DatacenterName

    "Remove Datacenter $Datacenter.Name"
    Remove-Datacenter -Datacenter $DatacenterName -Confirm:$false
}

Function New-VDPortgroupAndVDSwitch {
    <#
    .SYNOPSIS
        Create a new distributed port group with 1000 ports and add it to the new distributed switch
    .PARAMETER VDPortgroupName
        Specifies the name of the new distributed port group that you want to create
    .PARAMETER VDSwitchName
        Specifies a name for the new vSphere distributed switch that you want to create
    .PARAMETER DatacenterName
        Specifies the datacenter where you want to create the vSphere distributed switch
    .PARAMETER NumPorts
        Specifies the number of ports that the distributed port group will have
    .EXAMPLE
        New-VDPortgroupAndVDSwitch -VDPortgroupName 'theNewPortGroupName' -VDSwitchName 'theNewVDSwitch' -DatacenterName 'theDatacenter'
    .NOTES
        Copyright VMware, Inc. 2010-2013.  All Rights Reserved.
    #>

    Param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$VDPortgroupName,

        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$VDSwitchName,

        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$DatacenterName,
        
        [Parameter(Mandatory=$false)]
        [ValidateNotNullOrEmpty()]
        [string]$NumPorts = 1000
    )

    "Retrieve Datacenter $DatacenterName"
    $Datacenter = Get-Datacenter -Name $DatacenterName

    "Create a new vSphere distributed switch $VDSwitchName"
    $myVDSwitch = New-VDSwitch -Name $VDSwitchName -Location $Datacenter

    "Create a new distributed port group ($VDPortgroupName) with 1000 ports and add it to the distributed switch"
    New-VDPortgroup -Name "$VDPortgroupName" -VDSwitch $myVDSwitch -NumPorts $NumPorts
}

Function Remove-VMwareVDSwitch {
    <#
    .SYNOPSIS
         Removes vSphere distributed switch specified by the VDSwitchName parameter
    .PARAMETER VDSwitchName
        Specifies the vSphere distributed switches that you want to remove
    .EXAMPLE
        Remove-VDSwitch -VDSwitchName 'theDSwitch'
    .NOTES
        Copyright VMware, Inc. 2010-2013.  All Rights Reserved.
    #>

    Param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$VDSwitchName
    )

    "Removes vSphere distributed switch $VDSwitchName"
    Get-VDSwitch -Name $VDSwitchName | Remove-VDSwitch -Confirm:$false
}

Function Remove-VmwareVDPortgroup {
    <#
    .SYNOPSIS
        Removes distributed port groups specified by the VDPortgroupName parameter
    .PARAMETER VDPortgroupName
        Specifies the distributed port group that you want to remove
    .EXAMPLE
        Remove-VDPortgroup -VDPortgroupName 'thePortGroup'
    .NOTES
        Copyright VMware, Inc. 2010-2013.  All Rights Reserved.
    #>

    Param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$VDPortgroupName
    )

    "Removes distributed port group $VDPortgroupName"
    Get-VDPortGroup -Name $VDPortgroupName | Remove-VDPortGroup -Confirm:$false
}

