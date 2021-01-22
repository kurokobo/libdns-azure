# Azure DNS for `libdns`

This package implements the libdns interfaces for the [Azure DNS API](https://docs.microsoft.com/en-us/rest/api/dns/).

## Authenticating

This package supports authentication using Azure AD Application and Service Principal.

You will need to create a service principal using [PowerShell](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-authenticate-service-principal-powershell), [Azure Portal](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal), or [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli) and assign the following role to the DNS zones that you want to manage:

* DNS Zone Contributor

Then keep the following information to authenticate:

* `AZURE_TENANT_ID`
	* [Azure Active Directory] > [Properties] > [Tenant ID]
* `AZURE_CLIENT_ID`
	* [Azure Active Directory] > [App registrations] > Your Application > [Application ID]
* `AZURE_CLIENT_SECRET`
	* [Azure Active Directory] > [App registrations] > Your Application > [Certificates & secrets] > [Client secrets] > [Value]
* `AZURE_SUBSCRIPTION_ID`
	* [DNS zones] > Your Zone > [Subscription ID]
* `AZURE_RESOURCE_GROUP_NAME`
	* [DNS zones] > Your Zone > [Resource group]

## Example

Here's a minimal example of how to get all your DNS records using this `libdns` provider (see `_example/main.go`)

```go
package main

import (
	"context"
	"fmt"
	"os"
	"time"

	azure "github.com/kurokobo/libdns-azure"
	"github.com/libdns/libdns"
)

// main shows how libdns works with Azure DNS.
//
// To make this example work, you have to speficy some required environment variables:
// AZURE_TENANT_ID, AZURE_CLIENT_ID, AZURE_CLIENT_SECRET, AZURE_SUBSCRIPTION_ID,
// AZURE_RESOURCE_GROUP_NAME, AZURE_DNS_ZONE_FQDN
func main() {

	// Create new provider instance
	provider := azure.Provider{
		TenantId:          os.Getenv("AZURE_TENANT_ID"),
		ClientId:          os.Getenv("AZURE_CLIENT_ID"),
		ClientSecret:      os.Getenv("AZURE_CLIENT_SECRET"),
		SubscriptionId:    os.Getenv("AZURE_SUBSCRIPTION_ID"),
		ResourceGroupName: os.Getenv("AZURE_RESOURCE_GROUP_NAME"),
	}
	zone := os.Getenv("AZURE_DNS_ZONE_FQDN")

	// Invoke authentication and store client to instance
	if err := provider.NewClient(); err != nil {
		fmt.Printf("%v\n", err)
		return
	}

	// List existing records
	fmt.Printf("(1) List existing records\n")
	currentRecords, _ := provider.GetRecords(context.TODO(), zone)
	for _, record := range currentRecords {
		fmt.Printf("Exists: %v\n", record)
	}
}
```
