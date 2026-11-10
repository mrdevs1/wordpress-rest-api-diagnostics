# WordPress REST API Diagnostics

A practical guide to diagnosing, debugging, and troubleshooting WordPress REST API issues using HTTP requests, WP-CLI, PHP, and built-in WordPress functionality.

## What Is the WordPress REST API?

The WordPress REST API allows applications and external services to interact with WordPress through HTTP requests.

It supports operations such as:

- Retrieving posts and pages
- Creating and updating content
- Managing users and media
- Building headless WordPress applications
- Integrating third-party services
- Developing custom API endpoints

The default REST API namespace is:

```text
/wp-json/
```

## Common REST API Problems

- REST API returns HTTP 401 Unauthorized.
- API requests return HTTP 403 Forbidden.
- Custom endpoints return HTTP 404 Not Found.
- REST API requests fail with HTTP 500.
- Authentication fails for authenticated requests.
- Custom endpoint permissions are incorrect.
- API responses contain unexpected data.
- Security plugins or server rules block API requests.

## 1. Check Whether the REST API Is Accessible

Use curl to inspect the REST API index:

```bash
curl -i https://example.com/wp-json/
```

Check the WordPress posts endpoint:

```bash
curl -i https://example.com/wp-json/wp/v2/posts
```

A successful public endpoint request should generally return HTTP 200.

If the request fails, investigate permalink configuration, server rewrite rules, security restrictions, and plugin conflicts.

## 2. Inspect REST API HTTP Status Codes

HTTP status codes help identify the type of API failure.

| Status | Meaning | Common Cause |
|---|---|---|
| 200 | OK | Successful request |
| 201 | Created | Resource created |
| 400 | Bad Request | Invalid request parameters |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Invalid endpoint or resource |
| 405 | Method Not Allowed | Unsupported HTTP method |
| 500 | Internal Server Error | PHP or server-side error |

Always inspect the response body instead of relying only on the status code.

## 3. Debug REST API Responses

Use curl to display response headers and body:

```bash
curl -i https://example.com/wp-json/wp/v2/posts
```

For formatted JSON output:

```bash
curl -s https://example.com/wp-json/wp/v2/posts | python -m json.tool
```

For more detailed connection diagnostics:

```bash
curl -v https://example.com/wp-json/
```

This helps identify HTTP errors, redirects, SSL issues, and connection problems.

## 4. Troubleshoot REST API Authentication

WordPress supports different authentication approaches depending on the application and environment.

For authenticated requests from external applications, Application Passwords are available in supported WordPress installations.

Example:

```bash
curl -i -u "username:application-password" \
https://example.com/wp-json/wp/v2/users/me
```

Use an Application Password instead of the user's normal account password.

Important considerations:

- Use HTTPS.
- Never expose credentials in public repositories.
- Verify that the account has the required capabilities.
- Check whether security plugins block authentication requests.

## 5. Investigate Permission Callback Errors

Custom REST API endpoints should define an appropriate `permission_callback`.

Example:

```php
add_action('rest_api_init', function () {

    register_rest_route('custom/v1', '/reports', [
        'methods' => 'GET',

        'callback' => 'get_custom_reports',

        'permission_callback' => function () {
            return current_user_can('manage_options');
        },
    ]);

});

function get_custom_reports() {

    return [
        'message' => 'Reports retrieved successfully',
    ];

}
```

If a request returns HTTP 401 or 403, investigate authentication and authorization separately.

A permission callback returning `false` prevents access to the endpoint.

Do not use unrestricted permission callbacks for sensitive operations.

## 6. Debug Custom REST API Endpoints

List registered REST API routes:

```bash
wp rest route list
```

Inspect the registered route structure and verify:

- Namespace
- Endpoint path
- Supported HTTP methods
- Callback function
- Permission callback
- Expected request parameters

Example custom endpoint:

```php
add_action('rest_api_init', function () {

    register_rest_route('custom/v1', '/status', [
        'methods' => 'GET',

        'callback' => function () {
            return [
                'status' => 'active',
            ];
        },

        'permission_callback' => '__return_true',
    ]);

});
```

Test the endpoint:

```bash
curl -i https://example.com/wp-json/custom/v1/status
```

This example is intentionally public because it only returns non-sensitive status information.

## 7. Investigate REST API 404 Errors

If a valid endpoint returns HTTP 404, investigate:

1. Whether the REST API namespace is registered.
2. Whether the endpoint path is correct.
3. Whether the plugin registering the endpoint is active.
4. Whether permalink rewrite rules are working.
5. Whether server configuration blocks `/wp-json/`.

Flush rewrite rules when necessary:

```bash
wp rewrite flush
```

Avoid repeatedly flushing rewrite rules on production websites without a specific reason.

## 8. Investigate REST API 500 Errors

HTTP 500 generally indicates a server-side failure.

Enable debugging in a controlled environment:

```php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);
```

Check:

```text
wp-content/debug.log
```

Investigate:

- PHP fatal errors
- Invalid callback functions
- Missing dependencies
- Plugin conflicts
- Database errors
- Incorrect response handling

Disable debugging display and protect log files on production websites.

## 9. Validate REST API Request Parameters

Custom endpoints should validate incoming parameters.

Example:

```php
register_rest_route('custom/v1', '/product', [
    'methods' => 'GET',

    'callback' => 'get_product_data',

    'permission_callback' => '__return_true',

    'args' => [
        'id' => [
            'required' => true,

            'validate_callback' => function ($param) {
                return is_numeric($param) && (int) $param > 0;
            },

            'sanitize_callback' => 'absint',
        ],
    ],
]);
```

Parameter validation helps prevent invalid input from reaching application logic.

For write operations, also consider authorization, sanitization, and appropriate validation of every accepted field.

## 10. Troubleshoot REST API Conflicts

When an endpoint works in development but fails in production, investigate:

- Security and firewall rules
- Caching plugins
- CDN configuration
- Reverse proxy restrictions
- Incorrect server rewrite configuration
- Authentication headers removed by the server
- Plugin and theme conflicts

Test the endpoint with plugins disabled on a staging environment before changing production configuration.

## Essential REST API Diagnostic Commands

| Command | Purpose |
|---|---|
| `curl -i URL` | Inspect HTTP response |
| `curl -v URL` | Inspect connection details |
| `wp rest route list` | List registered REST routes |
| `wp plugin list` | Inspect installed plugins |
| `wp rewrite flush` | Refresh rewrite rules |
| `wp option get home` | Inspect WordPress home URL |
| `wp option get siteurl` | Inspect WordPress site URL |

## REST API Troubleshooting Checklist

- [ ] Verify `/wp-json/` accessibility.
- [ ] Check HTTP response status codes.
- [ ] Inspect API response headers and body.
- [ ] Verify authentication credentials.
- [ ] Check user capabilities.
- [ ] Review permission callbacks.
- [ ] Inspect registered REST API routes.
- [ ] Validate request parameters.
- [ ] Check PHP error logs.
- [ ] Investigate security and caching conflicts.
- [ ] Test changes on staging before production deployment.

## Important Security Guidelines

1. Always use HTTPS for authenticated API requests.
2. Never expose Application Passwords or authentication tokens.
3. Apply appropriate permission callbacks.
4. Validate and sanitize incoming parameters.
5. Avoid exposing sensitive information in API responses.
6. Use rate limiting where appropriate.
7. Do not disable security controls permanently to fix API errors.

## Conclusion

The WordPress REST API is an essential part of modern WordPress development.

Understanding authentication, permissions, HTTP responses, endpoint registration, and server configuration makes troubleshooting significantly easier.

This repository serves as a practical reference for WordPress developers working with REST API integrations and custom endpoints.
