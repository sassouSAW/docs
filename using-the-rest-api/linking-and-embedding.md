# Linking and Embedding

The WP REST API incorporates hyperlinking throughout the API to allow discoverability and browsability, as well as embedding related resources together in one response. While the REST API does not completely conform to the entire [HAL standard](http://stateless.co/hal_specification.html), it implements the `._links` and `._embedded` properties from that standard as described below.

## Links

The `_links` property of the response object contains a map of links to other API resources, grouped by "relation." The relation specifies how the linked resource relates to the primary resource. (Examples include "author," describing a relationship between a resource and its author, or "wp:term," describing the relationship between a post and its tags or categories.) The relation is either a [standardised relation](http://www.iana.org/assignments/link-relations/link-relations.xhtml#link-relations-1), a URI relation (like `https://api.w.org/term`) or a Compact URI relation (like `wp:term`). (Compact URI relations can be normalised to full URI relations to ensure full compatibility if required.) This is similar to HTML `&lt;link&gt;` tags, or `&lt;a rel=""&gt;` links.

The links are an object containing a `href` property with an absolute URL to the resource, as well as other optional properties. These include content types, disambiguation information, and data on actions that can be taken with the link.

For collection responses (those that return a list of objects rather than a top-level object), each item contains links, and the top-level response includes links via the `Link` header instead.

[info]
If your client library does not allow accessing headers, you can use the [`_envelope`](https://developer.wordpress.org/rest-api/global-parameters/#envelope) parameter to include the headers as body data instead.
[/info]

### Example Response
A typical single post request (`/wp/v2/posts/42`):

[javascript]
{
  &quot;id&quot;: 42,
  &quot;_links&quot;: {
    &quot;collection&quot;: [
      {
        &quot;href&quot;: &quot;https://demo.wp-api.org/wp-json/wp/v2/posts&quot;
      }
    ],
    &quot;author&quot;: [
      {
        &quot;href&quot;: &quot;https://demo.wp-api.org/wp-json/wp/v2/users/1&quot;,
        &quot;embeddable&quot;: true
      }
    ]
  }
}
[/javascript]

## Embedding
Optionally, some linked resources may be included in the response to reduce the number of HTTP requests required. These resources are "embedded" into the main response.

Embedding is triggered by setting the [`_embed` query parameter](https://developer.wordpress.org/rest-api/global-parameters/#embed) on the request. This will then include embedded resources under the `_embedded` key adjacent to the `_links` key. The layout of this object mirrors the `_links` object, but includes the embedded resource in place of the link properties.

Only links with the `embedded` flag set to `true` can be embedded, and `_embed` will cause all embeddable links to be embedded. Only relations containing embedded responses are included in `_embedded`, however relations with mixed embeddable and unembeddable links will contain dummy responses for the unembeddable links to ensure numeric indexes match those in `_links`.

### Example Response

[javascript]
{
  &quot;id&quot;: 42,
  &quot;_links&quot;: {
    &quot;collection&quot;: [
      {
        &quot;href&quot;: &quot;https://demo.wp-api.org/wp-json/wp/v2/posts&quot;
      }
    ],
    &quot;author&quot;: [
      {
        &quot;href&quot;: &quot;https://demo.wp-api.org/wp-json/wp/v2/users/1&quot;,
        &quot;embeddable&quot;: true
      }
    ]
  },
  &quot;_embedded&quot;: {
    &quot;author&quot;: {
      &quot;id&quot;: 1,
      &quot;name&quot;: &quot;admin&quot;,
      &quot;description&quot;: &quot;Site administrator&quot;
    }
  }
}
[/javascript]