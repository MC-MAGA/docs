# Google Cloud Storage

This page details on how to link your own Google Cloud Storage (GCS) bucket to Buildkite Package Registries, through a [private storage link](/docs/package-registries/private-storage-link).

By default, Buildkite Package Registries provides its own storage (called *Buildkite storage*). However, linking your own GCS bucket to Package Registries lets you:

- Keep packages and artifacts close to your geographical region for faster downloads.
- Retain full sovereignty over your packages and artifacts, while Buildkite continues to manage their metadata and indexing.

## Google Cloud's Workload Identity Federation feature

Each time Buildkite Package Registries uploads, downloads, or signs an object, the Buildkite platform presents its own OIDC token to Google Cloud's [Security Token Service](https://cloud.google.com/iam/docs/reference/sts/rest) (STS). The STS swaps this OIDC token for a short-lived access token representing your configured Google Cloud service account. This exchange is part of Google Cloud's [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation) feature, and allows Package Registries to perform its required actions inside your GCS bucket without the need for any storage of long-lived credentials.

Key security benefits of Workload Identity Federation:

- No long-lived [service account keys](https://cloud.google.com/iam/docs/service-account-creds#key-types)—tokens are generated when they're required and expire automatically.

- Least-privilege access—Buildkite Package Registries only requires these Google Cloud Identity and Access Management (IAM) roles:
    * `roles/storage.bucketViewer` on the GCS bucket, to allow Package Registries to read the bucket's metadata.
    * `roles/storage.objectUser` on the GCS bucket to allow Package Registries to create, read, update, delete, and tag objects.
    * `roles/iam.serviceAccountTokenCreator` on the Google Cloud service account so it can mint signed URLs.
You can audit or revoke these at any time.

- Full audit history—every token exchange and object access is recorded in Google Cloud service's audit logs.

## Before you start

Before you begin, you'll need a GCS bucket in a Google Cloud project that Buildkite can access.

Learn more about:

- Google Cloud Storage from its main [Cloud Storage](https://cloud.google.com/storage) page.

- How to create a GCS bucket from Google's [Create a bucket](https://cloud.google.com/storage/docs/creating-buckets) guide.
    * As part of creating your GCS bucket, learn more about the [requirements for naming your bucket](https://cloud.google.com/storage/docs/naming-buckets), especially for creating ones with globally-unique names.

- How to keep your GCS bucket private by default from Google's [Public access prevention](https://cloud.google.com/storage/docs/public-access-prevention) guide.

- How to protect again accidental object deletion from Google's [Object Versioning](https://cloud.google.com/storage/docs/object-versioning) guide.

Once you have a bucket, the Buildkite wizard will guide you through:

1. **Creating (or selecting) a service account** for Buildkite to impersonate.
1. **Creating a Workload Identity _Pool_ and _Provider_**.
1. **Granting IAM roles** to wire everything together.

## Link your private GCS bucket to Buildkite Package Registries

To link your private Google Cloud Storage (GCS) bucket to Package Registries:

1. As a [Buildkite organization administrator](/docs/package-registries/security/permissions#manage-teams-and-permissions-organization-level-permissions), select **Settings** in the global navigation to open the [**Organization Settings**](https://buildkite.com/organizations/~/settings) page.

1. In the **Packages** section, select **Private Storage Link** to open its page.

1. Click **Add private storage link** to start the private storage configuration process.

1. On the **Provide your storage's details** (page) > **Step 1: Select your storage provider**, select **GCS**.

1. In **Step 2: Create or locate your Google Cloud bucket**, select **Open Google Cloud** to open the list of GCS buckets in your Google Cloud account, to either retrieve your existing empty GCS bucket, or create a new one if you [haven't already done so](#before-you-start).

    **Notes:**
    * If you are already familiar with working with Google Cloud Storage and need to create a new GCS bucket, expand the **Create a new bucket** section for quick instructions on this process.
    * Ensure you are in the correct Google Cloud _organization_ and _project_ in which to create your GCS bucket.

1. Back on the Buildkite interface, in **Step 3: Enter your Google Cloud bucket details**, specify the **Bucket name** (for example, `my-gcs-bucket`) for your GCS bucket, then select **Continue**.

1. On the **Connect Buildkite to Google** page, access and follow the relevant instructions to either:
    * Create a new Google Cloud (GC) service account and Workload Identity Pool and Provider (WIPP) using the Google Cloud Console or CLI—expand the relevant **Create new** section. Links to the relevant Google Cloud Console pages are provided.
    * Use an existing GC service account and WIPP—expand the relevant **Find existing** section.

1. From your Google Cloud Console or CLI output of creating a new or finding an existing GC service account, copy the **Email** address for this account (for example, `buildkite-storage-link@my-google-cloud-project.iam.gserviceaccount.com`), and paste the value into the **Service account email** field on the **Connect Buildkite to Google** Package Registries page.

1. From your Google Cloud Console or CLI output of creating a new or finding an existing WIPP, copy the path component (`projects/<project-number>/locations/global/workloadIdentityPools/<pool>/providers/<provider>`) of the URL within **Audiences** > **Default audience** (when creating or editing the workload identity provider), and paste this path component value into the **Workload Identity Provider (full resource name)** field on the **Connect Buildkite to Google** Package Registries page.

1. Select **Next**.

1. On the next **Connect Buildkite to Google** page's **Allow Buildkite to impersonate service account** section, follow the instructions to allow the Buildkite platform to impersonate your GC service account (through the **Grant access using service account impersonation** option).

    **Note:** If you are using the Google Cloud Console to implement this configuration, then the **Workload Identity User** (`roles/iam.workloadIdentityUser`) role is automatically applied to the GC service account as part of this process.

1. In the next **Grant bucket access** section, follow the instructions grant the service account `roles/storage.objectUser` and `roles/storage.bucketViewer` on your bucket so Buildkite can manage package objects and read bucket metadata.

1. Select **Run diagnostic**. Buildkite uploads, downloads, and tags a test object to confirm it can:
    * publish (`PUT`)
    * download (`GET`)
    * generate a signed URL (`signBlob`)
    * tag with metadata (to allow lifecycle rules to delete)
    * delete (`DELETE`)

1. When all tests pass, click **Create Private Storage Link** to finish.

You're returned to the **Private Storage Link** page where you can:

- [Set the default Buildkite Package Registries storage for your organization](/docs/package-registries/private-storage-link#set-the-default-buildkite-package-registries-storage).
- [Choose storage per registry](/docs/package-registries/manage-registries#update-a-source-registry-configure-registry-storage).

## Set as default storage (optional)

Once at least one PSL exists you can change the organization’s default storage (this affects newly-created registries):

1. **Organization Settings > Packages > Private Storage Link**.
1. Click **Change** and select the `gs://…` link.

## Deleting packages

Buildkite does **not** delete objects straight away. When a package is removed it:

- Adds a metadata tag of `buildkite:deleted=<ISO8601_timestamp>`.
- Sets the object’s `customTime` field to the same timestamp.

| Metadata key       | Value (UTC) |
|--------------------|-------------|
| `buildkite:deleted`| ISO 8601 timestamp |

Create a [Lifecycle rule](https://cloud.google.com/storage/docs/lifecycle) that removes objects a set number of days after their `customTime`. For example, to purge 30 days after deletion:

```json
{
  "rule": [
    {
      "action": { "type": "Delete" },
      "condition": { "daysSinceCustomTime": 30 }
    }
  ]
}
```
