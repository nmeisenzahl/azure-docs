---
title: Connect privately to a Web App using Azure Private Endpoint
description: Connect privately to a Web App using Azure Private Endpoint
author: ericgre
ms.assetid: 2dceac28-1ba6-4904-a15d-9e91d5ee162c
ms.topic: article
ms.date: 03/12/2020
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.custom: fasttrack-edit

---

# Using Private Endpoints for Azure Web App (Preview)

> [!Note]
> The preview is available in East US region for all PremiumV2 Windows and Linux Web Apps and Elastic Premium Functions. 

You can use Private Endpoint for your Azure Web App to allow clients located in your private network to securely access the app over Private Link. The Private Endpoint uses an IP address from your Azure VNet address space. Network traffic between a client on your private network and the Web App traverses over the VNet and a Private Link on the Microsoft backbone network, eliminating exposure from the public Internet.

Using Private Endpoint for your Web App enables you to:

- Secure your Web App by configuring the Service Endpoint, eliminating public exposure.
- Securely connect to Web App from on-premises networks that connect to the VNet using a VPN or ExpressRoute private peering.

If you just need a secure connection between your VNet and your Web App, a Service Endpoint is the simplest solution. If you also need to reach the web app from on-premises through an Azure gateway, a regionally peered VNet or a globally peered VNet, Private Endpoint is the solution.  

For more information, see [Service Endpoints][serviceendpoint].

## Conceptual overview

A Private Endpoint is a special network interface (NIC) for your Azure Web App in a Subnet in your Virtual Network (VNet).
When you create a Private Endpoint for your Web App, it provides secure connectivity between clients on your private network and your Web App. The Private Endpoint is assigned an IP Address from the IP address range of your VNet.
The connection between the Private Endpoint and the Web App uses a secure [Private Link][privatelink]. Private Endpoint is only used for incoming flows to your Web App. Outgoing flows will not use this Private Endpoint, but you can inject outgoing flows to your network in a different subnet through the [VNet integration feature][vnetintegrationfeature].

The Subnet where you plug the Private Endpoint can have other resources in it, you don't need a dedicated empty Subnet.
You can also deploy the Private Endpoint in a different region than the Web App. 

> [!Note]
>The VNet integration feature cannot use the same subnet than Private Endpoint, this is a limitation of the VNet integration feature.

From a security perspective:

- When you enable Private Endpoints to your Web App, you disable all public access.
- You can enable multiple Private Endpoints in others VNets and Subnets, including VNets in other regions.
- The IP address of the Private Endpoint NIC must be dynamic, but will remain the same until you delete the Private Endpoint.
- The NIC of the Private Endpoint cannot have an NSG associated.
- The Subnet that hosts the Private Endpoint can have an NSG associated, but you must disable the network policies enforcement for the Private Endpoint: see [Disable network policies for private endpoints][disablesecuritype]. As a result, you cannot filter by any NSG the access to your Private Endpoint.
- When you enable Private Endpoint to your Web App, the [access restrictions][accessrestrictions] configuration of the Web App is not evaluated.
- You can reduce the data exfiltration risk from the VNet by removing all NSG rules where destination is tag Internet or Azure services. But adding a Web App Service Endpoint in your subnet will let you reach any Web App hosted in the same deployment stamp and exposed to the Internet.

In the Web HTTP logs of your Web App, you will find the client source IP. This is implemented using the TCP Proxy protocol, forwarding the client IP property up to the Web App. For more information, see [Getting connection Information using TCP Proxy v2][tcpproxy].

![Global overview][1]


## DNS

As this feature is in preview, we don't change the DNS entry during the preview. You need to manage the DNS entry in your private DNS server or Azure DNS private zone yourself.
If you need to use a custom DNS name, you must add the custom name in your Web App. During the preview, the custom name must be validated like any custom name, using public DNS resolution. See [custom DNS validation][dnsvalidation] for more information.

## Pricing

For pricing details, see [Azure Private Link pricing][pricing].

## Limitations

We are improving Private Link feature and Private Endpoint regularly, check [this article][pllimitations] for up-to-date information about limitations.

## Next steps

To deploy Private endpoint for your Web App through the portal see [How to connect privately to a Web App][howtoguide]


<!--Image references-->
[1]: ./media/private-endpoint/schemaglobaloverview.png

<!--Links-->
[serviceendpoint]: https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview
[privatelink]: https://docs.microsoft.com/azure/private-link/private-link-overview
[vnetintegrationfeature]: https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet
[disablesecuritype]: https://docs.microsoft.com/azure/private-link/disable-private-endpoint-network-policy
[accessrestrictions]: https://docs.microsoft.com/azure/app-service/app-service-ip-restrictions
[tcpproxy]: ../../private-link/private-link-service-overview.md#getting-connection-information-using-tcp-proxy-v2
[dnsvalidation]: https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain
[pllimitations]: https://docs.microsoft.com/azure/private-link/private-endpoint-overview#limitations
[pricing]: https://azure.microsoft.com/pricing/details/private-link/
[howtoguide]: https://docs.microsoft.com/azure/private-link/create-private-endpoint-webapp-portal
