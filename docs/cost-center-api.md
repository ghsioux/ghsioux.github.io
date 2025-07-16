Improvement
June 30, 2025 • 1 minute read
Managing cost centers via API is now available

Summary
Enterprise customers can now manage cost centers and assign resources via API

Starting today, Enterprise customers can manage the full lifecycle of cost centers using the REST API. With the improved API, you can create, edit, and delete cost centers. You can also programmatically add, remove, and reassign resources (e.g., organizations, repositories, and users) from cost centers.

This release also improves API performance and addresses top customer feedback, including reflecting the cost center state in the API response.

For more details, refer to REST API endpoints for cost centers.

Join the discussion within GitHub Community.



You can use the REST API to manage cost centers. The API allows you to create and delete cost centers, as well as add or remove resources from them.

The authenticated user must be an enterprise admin to use these endpoints.

## Creating a cost center

```text
POST /enterprises/{enterprise}/settings/billing/cost-centers
```

Creates a new cost center for an enterprise.

### Headers

| Name            | Type   | Description                                                    |
| --------------- | ------ | -------------------------------------------------------------- |
| `accept`        | string | Setting to `application/vnd.github+json` is recommended.       |
| `authorization` | string | Required. A personal access token with the appropriate scopes. |

### Path parameters

| Name         | Type   | Required | Description                                                                                         |
| ------------ | ------ | -------- | --------------------------------------------------------------------------------------------------- |
| `enterprise` | string | Yes      | The slug version of the enterprise name. You can also substitute this value with the enterprise id. |

### Query parameters

| Name   | Type   | Description                                                  |
| ------ | ------ | ------------------------------------------------------------ |
| `name` | string | The name of the cost center. Maximum length: 255 characters. |

#### Example request

```json
{
  "name": "Engineering Team"
}
```

### HTTP response status codes for "Creating a cost center"

| Status code | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `200`       | Cost center created successfully                             |
| `400`       | Bad request (e.g., missing required fields or limit reached) |
| `409`       | Conflict (e.g., duplicate cost center name)                  |
| `500`       | Internal server error                                        |

### Example response

Status: 200

```json
{
  "id": "abc123",
  "name": "Engineering Team",
  "resources": []
}
```

### Code Samples

#### cURL

```bash
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/enterprises/ENTERPRISE/settings/billing/cost-centers \
  -d '{"name":"Engineering Team"}'
```

#### JavaScript

```javascript
// Octokit.js
// https://github.com/octokit/core.js#readme
const octokit = new Octokit({
  auth: 'YOUR-TOKEN'
})

await octokit.request('POST /enterprises/{enterprise}/settings/billing/cost-centers', {
  enterprise: 'ENTERPRISE',
  name: 'Engineering Team',
  headers: {
    'X-GitHub-Api-Version': '2022-11-28'
  }
})
```

#### Creating a cost center using GitHub CLI

```bash
gh api \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  /enterprises/ENTERPRISE/settings/billing/cost-centers \
  -f name='Engineering Team'
```

## Adding resources to a cost center

```text
POST /enterprises/{enterprise}/settings/billing/cost-centers/{cost_center_id}/resource
```

Adds organizations, repositories, or users to a cost center. Added resources' usage will be charged to the cost center's budget.

### Headers

| Name     | Type   | Description                                              |
| -------- | ------ | -------------------------------------------------------- |
| `accept` | string | Setting to `application/vnd.github+json` is recommended. |

### Path parameters

| Name             | Type   | Required | Description                                                                                         |
| ---------------- | ------ | -------- | --------------------------------------------------------------------------------------------------- |
| `enterprise`     | string | Yes      | The slug version of the enterprise name. You can also substitute this value with the enterprise id. |
| `cost_center_id` | string | Yes      | The ID of the cost center to which you want to add resources.                                       |

### Query parameters

| Name            | Type             | Description                                  |
| --------------- | ---------------- | -------------------------------------------- |
| `organizations` | array of strings | The organizations to add to the cost center. |
| `repositories`  | array of strings | The repositories to add to the cost center.  |
| `users`         | array of strings | The users to add to the cost center.         |

#### Example request

```json
{
  "organizations": ["my-org"],
  "repositories": ["my-org/my-repo"],
  "users": ["jdoe", "janedoe2"]
}
```

### HTTP response status codes for "Adding organizations, repositories, or users to a cost center"

| Status code | Description                  |
| ----------- | ---------------------------- |
| `200`       | Resources added successfully |
| `400`       | Bad request                  |
| `403`       | Forbidden                    |
| `409`       | Conflict                     |
| `500`       | Internal server error        |
| `503`       | Service unavailable          |

### Code Samples

#### Adding resources using REST API (cURL)

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/enterprises/{enterprise}/settings/billing/cost-centers/{costCenterID}/resource \
  -d '{
    "organizations": ["my-org"],
    "repositories": ["my-org/my-repo"],
    "users": ["username"]
  }'
```

#### Adding resources using GitHub CLI

```bash
gh api \
  --method POST \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  /enterprises/{enterprise}/settings/billing/cost-centers/{costCenterID}/resource \
  -f organizations='["my-org"]' \
  -f repositories='["my-org/my-repo"]' \
  -f users='["username"]'
```

#### Example response

```json
{
  "message": "Resources successfully added to the cost center."
}
```

## Removing resources from cost centers

You can remove organizations, repositories, and users from a cost center. Removed resources' charges will revert to your enterprise or organization's default billing method.

To remove a resource from a cost center, use the following endpoint:

```text
DELETE /enterprises/{enterprise}/settings/billing/cost-centers/{cost_center_id}/resource
```

### Headers

| Name     | Type   | Description                                              |
| -------- | ------ | -------------------------------------------------------- |
| `accept` | string | Setting to `application/vnd.github+json` is recommended. |

### Path parameters

| Name             | Type   | Required | Description                                                                                         |
| ---------------- | ------ | -------- | --------------------------------------------------------------------------------------------------- |
| `enterprise`     | string | Yes      | The slug version of the enterprise name. You can also substitute this value with the enterprise id. |
| `cost_center_id` | string | Yes      | The ID of the cost center from which you want to remove resources.                                  |

### Request body

Provide one or more of the following fields with the resources you wish to remove:

| Name            | Type             | Description                                       |
| --------------- | ---------------- | ------------------------------------------------- |
| `organizations` | array of strings | The organizations to remove from the cost center. |
| `repositories`  | array of strings | The repositories to remove from the cost center.  |
| `users`         | array of strings | The users to remove from the cost center.         |

#### Example request

```json
{
  "organizations": ["my-org"],
  "repositories": ["my-org/my-repo"],
  "users": ["jdoe", "janedoe2"]
}
```

### HTTP response status codes

| Status code | Description                    |
| ----------- | ------------------------------ |
| `200`       | Resources removed successfully |
| `400`       | Bad request                    |
| `403`       | Forbidden                      |
| `409`       | Conflict                       |
| `500`       | Internal server error          |
| `503`       | Service unavailable            |

### Code samples

#### Removing a resource from a cost center using REST API

```bash
curl -X DELETE \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/enterprises/{enterprise}/settings/billing/cost-centers/{costCenterID}/resource \
  -d '{"organizations":["org-to-remove"],"repositories":["repo-to-remove"]}'
```

#### Removing a resource from a cost center using GitHub CLI

```bash
gh api \
  --method DELETE \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  /enterprises/{enterprise}/settings/billing/cost-centers/{costCenterID}/resource \
  -f organizations='["org-to-remove"]' \
  -f repositories='["repo-to-remove"]'
```

#### Example response

```json
{
  "message": "Resource successfully removed from cost center"
}
```

## Deleting a cost center

### Headers

| Name            | Type   | Description                                                    |
| --------------- | ------ | -------------------------------------------------------------- |
| `accept`        | string | Setting to `application/vnd.github+json` is recommended.       |
| `authorization` | string | Required. A personal access token with the appropriate scopes. |

### Path parameters

| Name             | Type   | Required | Description                                                                                         |
| ---------------- | ------ | -------- | --------------------------------------------------------------------------------------------------- |
| `enterprise`     | string | Yes      | The slug version of the enterprise name. You can also substitute this value with the enterprise id. |
| `cost_center_id` | string | Yes      | The ID of the cost center to delete.                                                                |

### HTTP response status codes for "Delete a cost center"

| Status code | Description                      |
| ----------- | -------------------------------- |
| `200`       | Cost center deleted successfully |
| `400`       | Bad request                      |
| `403`       | Forbidden                        |
| `404`       | Cost center not found            |
| `500`       | Internal server error            |
| `503`       | Service unavailable              |

### Code Samples

#### cURL

```bash
curl -L \
  -X DELETE \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/enterprises/ENTERPRISE/settings/billing/cost-centers/COST_CENTER_ID

```

#### JavaScript

```javascript
// Octokit.js
// https://github.com/octokit/core.js#readme
const octokit = new Octokit({
  auth: 'YOUR-TOKEN'
})

await octokit.request('DELETE /enterprises/{enterprise}/settings/billing/cost-centers/{cost_center_id}', {
  enterprise: 'ENTERPRISE',
  cost_center_id: 'COST_CENTER_ID',
  headers: {
    'X-GitHub-Api-Version': '2022-11-28'
  }
})
```

#### Deleting a cost center using GitHub CLI

```bash
gh api \
  -X DELETE \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  /enterprises/ENTERPRISE/settings/billing/cost-centers/COST_CENTER_ID
```