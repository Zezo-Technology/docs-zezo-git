---
title: Combined GitHub Enterprise cloud and server use
intro: 'Your enterprise account enables you to set up {% data variables.product.prodname_ghe_server %} with no additional cost.'
versions:
  fpt: '*'
  ghec: '*'
  ghes: '*'
topics:
  - Billing
  - Enterprise
  - Licensing
shortTitle: Combined enterprise use
redirect_from:
  - /billing/managing-your-license-for-github-enterprise/about-licenses-for-github-enterprise
contentType: concepts
---


{% data reusables.billing.usage-based-billing %}

## About licensing for {% data variables.product.prodname_enterprise %}

{% data reusables.enterprise.about-deployment-methods %}

{% data reusables.enterprise-licensing.unique-user-licensing-model %} To understand how {% data variables.product.company_short %} bills you for consumed {% ifversion enterprise-licensing-language %}licenses{% else %}licensed seats{% endif %}, see [AUTOTITLE](/billing/managing-the-plan-for-your-github-account/about-per-user-pricing). For more about the price of {% data variables.product.prodname_enterprise %} licenses, see [Pricing](https://github.com/pricing) on the {% data variables.product.prodname_dotcom %} website.

To ensure the same user isn't consuming more than one license for multiple enterprise deployments, you can synchronize license usage between your {% data variables.product.prodname_ghe_server %} and {% data variables.product.prodname_ghe_cloud %} deployments.

{% ifversion ghes %}

{% data reusables.billing.license-type-overview %}
{% data reusables.billing.license-models %}

{% endif %}

In order to use a {% data variables.product.prodname_ghe_server %} instance, you must upload a license file that {% data variables.product.company_short %} provides when you purchase, renew, or add user licenses to {% data variables.product.prodname_enterprise %}.

{% data variables.product.prodname_ghe_cloud %} customers can use Azure DevOps without additional per user charges. Access is available to customers using Microsoft Entra and users logging in with the same credentials in {% data variables.product.github %} and Azure DevOps.

There is no action required; {% data variables.product.prodname_enterprise %} users will be detected automatically when they log in to Azure DevOps. See [User and permissions management](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/faq-user-and-permissions-management?view=azure-devops#github-enterprise) in the Microsoft Learn documentation.

Access is not yet available in [new data residency regions](https://github.blog/engineering/engineering-principles/github-enterprise-cloud-with-data-residency/) but will be coming in the future.

## About synchronization of license usage for {% data variables.product.prodname_enterprise %}

{% data reusables.enterprise-licensing.about-license-sync %} For more information, see [AUTOTITLE](/billing/managing-your-license-for-github-enterprise/syncing-license-usage-between-github-enterprise-server-and-github-enterprise-cloud).

## About license files for {% data variables.product.prodname_enterprise %}

When you purchase or renew {% data variables.product.prodname_enterprise %}, {% data variables.product.company_short %} provides a license file {% ifversion ghec %}for your deployments of {% data variables.product.prodname_ghe_server %}{% elsif ghes %}for {% data variables.location.product_location_enterprise %}{% endif %}. A license file has an expiration date and controls the number of people who can use {% data variables.location.product_location_enterprise %}. After you download and install {% data variables.product.prodname_ghe_server %}, you must upload the license file to unlock the application for you to use.

For more information about downloading your license file, see [AUTOTITLE](/billing/managing-your-license-for-github-enterprise/downloading-your-license-for-github-enterprise).

For more information about uploading your license file, see {% ifversion ghec %}[AUTOTITLE](/enterprise-server@latest/billing/managing-your-license-for-github-enterprise/uploading-a-new-license-to-github-enterprise-server) in the {% data variables.product.prodname_ghe_server %} documentation.{% elsif ghes %}[AUTOTITLE](/billing/managing-your-license-for-github-enterprise/uploading-a-new-license-to-github-enterprise-server).{% endif %}

If your license expires, you won't be able to access {% data variables.product.prodname_ghe_server %} via a web browser or Git. If needed, you will be able to use command-line utilities to back up all your data. For more information, see {% ifversion ghec %}[Configuring backups on your appliance](/enterprise-server@latest/admin/guides/installation/configuring-backups-on-your-appliance) in the {% data variables.product.prodname_ghe_server %} documentation.{% elsif ghes %}[AUTOTITLE](/admin/configuration/configuring-your-enterprise/configuring-backups-on-your-appliance).{% endif %}

If you have any questions about renewing your license, contact {% data variables.contact.contact_enterprise_sales %}.

## Further reading

* [AUTOTITLE](/billing/managing-your-billing/about-billing-for-your-enterprise)
* The [People that consume a license](/billing/managing-the-plan-for-your-github-account/about-per-user-pricing#people-that-consume-a-license) section in "About per-user pricing"
* [AUTOTITLE]({% ifversion fpt or ghec %}/enterprise-server@latest{% endif %}/admin/installation/setting-up-a-github-enterprise-server-instance)
* The [{% data variables.product.prodname_enterprise %} Releases](https://enterprise.github.com/releases/) website
