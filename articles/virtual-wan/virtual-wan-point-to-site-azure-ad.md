---
title: 'Configure a User VPN connection using Azure Active Directory authentication'
titleSuffix: Azure Virtual WAN
description: Learn how to configure Azure Active Directory authentication for Virtual WAN User VPN (point-to-site).
services: virtual-wan
author: cherylmc

ms.service: virtual-wan
ms.topic: how-to
ms.date: 08/19/2021
ms.author: cherylmc 
ms.custom: devx-track-azurepowershell

---
# Configure a User VPN connection - Azure Active Directory authentication

This article shows you how to configure Azure AD authentication for User VPN in Virtual WAN to connect to your resources in Azure over an OpenVPN VPN connection. Azure Active Directory authentication is only available for gateways using OpenVPN protocol and clients running Windows.

This type of connection requires a client to be configured on the client computer. For more information about Virtual WAN, see the [Virtual WAN Overview](virtual-wan-about.md).

In this article, you learn how to:

* Create a Virtual WAN
* Create a Virtual Hub
* Create a User VPN configuration
* Download a Virtual WAN User VPN profile
* Apply User VPN configuration to a Virtual Hub
* Connect a VNet to a Virtual Hub
* Download and apply the User VPN client configuration
* View your Virtual WAN

![Virtual WAN diagram](./media/virtual-wan-about/virtualwanp2s.png)

## Before you begin

Verify that you have met the following criteria before beginning your configuration:

* You have a virtual network that you want to connect to. Verify that none of the subnets of your on-premises networks overlap with the virtual networks that you want to connect to. To create a virtual network in the Azure portal, see the [Quickstart](../virtual-network/quick-create-portal.md).

* Your virtual network does not have any virtual network gateways. If your virtual network has a gateway (either VPN or ExpressRoute), you must remove all gateways. This configuration requires that virtual networks are connected instead, to the Virtual WAN hub gateway.

* Obtain an IP address range for your hub region. The hub is a virtual network that is created and used by Virtual WAN. The address range that you specify for the hub cannot overlap with any of your existing virtual networks that you connect to. It also cannot overlap with your address ranges that you connect to on premises. If you are unfamiliar with the IP address ranges located in your on-premises network configuration, coordinate with someone who can provide those details for you.

* If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="wan"></a>Create a Virtual WAN

From a browser, navigate to the [Azure portal](https://portal.azure.com) and sign in with your Azure account.

1. Navigate to the Virtual WAN page. In the portal, click **+Create a resource**. Type **Virtual WAN** into the search box and select Enter.
2. Select **Virtual WAN** from the results. On the Virtual WAN page, click **Create** to open the Create WAN page.
3. On the **Create WAN** page, on the **Basics** tab, fill in the following fields:

   ![Virtual WAN](./media/virtual-wan-point-to-site-azure-ad/vwan.png)

   * **Subscription** - Select the subscription that you want to use.
   * **Resource group** - Create new or use existing.
   * **Resource group location** - Choose a resource location from the dropdown. A WAN is a global resource and does not live in a particular region. However, you must select a region in order to more easily manage and locate the WAN resource that you create.
   * **Name** - Type the Name that you want to call your WAN.
   * **Type:** Standard. If you create a Basic WAN, you can create only a Basic hub. Basic hubs are capable of VPN site-to-site connectivity only.
4. After you finish filling out the fields, select **Review +Create**.
5. After validation passes, select **Create** to create the virtual WAN.

## <a name="site"></a>Create an empty Virtual hub

1. Under your Virtual WAN, select Hubs and click **+New Hub**.

   ![Screenshot shows the Hubs configuration dialog box with New Hub selected.](media/virtual-wan-point-to-site-azure-ad/hub1.jpg)
2. On the create virtual hub page, fill in the following fields.

   **Region** - Select the region that you want to deploy the virtual hub in.

   **Name** - Enter the name that you want to call your virtual hub.

   **Hub private address space** - The hub's address range in CIDR notation.

   ![Screenshot shows the Create virtual hub pane where you can enter values.](media/virtual-wan-point-to-site-azure-ad/hub2.jpg)  
3. Click **Review + create**.
4. On the **validation passed** page, click **create**.

## <a name="site"></a>Create a new User VPN configuration

A User VPN configuration defines the parameters for connecting remote clients.

1. Under your virtual WAN, select **User VPN configurations**.

   ![Screenshot shows the User V P N configurations menu item selected.](media/virtual-wan-point-to-site-azure-ad/aadportal1.jpg)

2. Click **+Create user VPN config**.

   ![Screenshot shows the Create user V P N config link.](media/virtual-wan-point-to-site-azure-ad/aadportal2.jpg)

3. Under **Basics**, specify the following parameters
   * **Configuration name** - Enter the name you want to call your User VPN Configuration.
    * **Tunnel type** - Select OpenVPN from the dropdown menu.
4. Navigate to **Azure Active Directory**. Toggle **Azure Active Directory** to 'Yes' and supply the following values based on your tenant details. 
   * **Authentication method** - Select Azure Active Directory.
   * **Audience** - Type in the Application ID of the [Azure VPN](openvpn-azure-ad-tenant.md) Enterprise Application registered in your Azure AD tenant. 
   * **Issuer** - `https://sts.windows.net/<your Directory ID>/`
   * **AAD Tenant** - `https://login.microsoftonline.com/<your Directory ID>`
  
   ![Screenshot shows the Create new User V P N configuration pane where you can enter the values.](media/virtual-wan-point-to-site-azure-ad/configure-aad-profile.png)

## <a name="hub"></a>Edit hub assignment

1. Navigate to the **Hubs** blade under the virtual WAN.
2. Select the hub that you want to associate the vpn server configuration to and click the ellipsis (...).

   ![Screenshot shows Edit virtual hub selected from the menu.](media/virtual-wan-point-to-site-azure-ad/select-hub.png)
3. Click **Edit virtual hub**.
4. Check the **Include point-to-site gateway** check box and pick the **Gateway scale unit** that you want.

   :::image type="content" source="./media/virtual-wan-point-to-site-azure-ad/edit-virtual-hub.png" alt-text="Screenshot shows the Edit virtual hub dialog box where you can select your Gateway scale unit."lightbox="./media/virtual-wan-point-to-site-azure-ad/edit-virtual-hub.png":::

5. Enter the **Address pool** from which the VPN clients will be assigned IP addresses.
6. Click **Confirm**.
7. The operation will can take up to 30 minutes to complete.

## <a name="device"></a>Download User VPN profile

Use the VPN profile to configure your clients.

1. On the page for your Virtual WAN, click **User VPN configurations**.
2. At the top of the  page, click **Download user VPN config**.
3. Once the file has finished creating, you can click the link to download it.
4. Use the profile file to configure the VPN clients.

## Configure user VPN clients

To connect, you need to download the Azure VPN Client and import the VPN client profile that was downloaded in the previous steps on every computer that wants to connect to the VNet.

> [!NOTE]
> Azure AD authentication is supported only for OpenVPN&reg; protocol connections.
>

#### To download the Azure VPN client

Use this [link](https://www.microsoft.com/p/azure-vpn-client-preview/9np355qt2sqb?rtc=1&activetab=pivot:overviewtab) to download the Azure VPN Client.

#### <a name="import"></a>To import a client profile

1. On the page, select **Import**.

    ![Screenshot shows Import selected from the plus menu.](./media/virtual-wan-point-to-site-azure-ad/import/import1.jpg)

2. Browse to the profile xml file and select it. With the file selected, select **Open**.

    ![Screenshot shows an Open dialog box where you can select a file.](./media/virtual-wan-point-to-site-azure-ad/import/import2.jpg)

3. Specify the name of the profile and select **Save**.

    ![Screenshot shows the Connection Name added and the Save button selected.](./media/virtual-wan-point-to-site-azure-ad/import/import3.jpg)

4. Select **Connect** to connect to the VPN.

    ![Screenshot shows the Connect button for the for the connection you just created.](./media/virtual-wan-point-to-site-azure-ad/import/import4.jpg)

5. Once connected, the icon will turn green and say **Connected**.

    ![Screenshot shows the connection in a Connected status with the option to disconnect.](./media/virtual-wan-point-to-site-azure-ad/import/import5.jpg)

#### <a name="delete"></a>To delete a client profile

1. Select the ellipsis (...) next to the client profile that you want to delete. Then, select **Remove**.

    ![Screenshot shows Remove selected from the menu.](./media/virtual-wan-point-to-site-azure-ad/delete/delete1.jpg)

2. Select **Remove** to delete.

    ![Screenshot shows a confirmation dialog box with the option to Remove or Cancel.](./media/virtual-wan-point-to-site-azure-ad/delete/delete2.jpg)

#### <a name="diagnose"></a>Diagnose connection issues

1. To diagnose connection issues, you can use the **Diagnose** tool. Select the ellipsis (...) next to the VPN connection that you want to diagnose to reveal the menu. Then select **Diagnose**.

    ![Screenshot shows Diagnose selected from the menu.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose1.jpg)

2. On the **Connection Properties** page, select **Run Diagnosis**.

    ![Screenshot shows the Run Diagnosis button for a connection.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose2.jpg)

3. Sign in with your credentials.

    ![Screenshot shows the Sign in dialog box for this action.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose3.jpg)

4. View the diagnosis results.

    ![Screenshot shows the results of the diagnosis.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose4.jpg)

## <a name="viewwan"></a>View your virtual WAN

1. Navigate to the virtual WAN.
2. On the Overview page, each point on the map represents a hub.
3. In the Hubs and connections section, you can view hub status, site, region, VPN connection status, and bytes in and out.

## <a name="cleanup"></a>Clean up resources

When you no longer need these resources, you can use [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) to remove the resource group and all of the resources it contains. Replace "myResourceGroup" with the name of your resource group and run the following PowerShell command:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## Next steps

To learn more about Virtual WAN, see the [Virtual WAN Overview](virtual-wan-about.md) page.
