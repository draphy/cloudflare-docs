---
pcx_content_type: concept
title: Redirects
---

# Redirects

To use redirects on Cloudflare Pages, declare your redirects in a plain text file called `_redirects` without a file extension, in the output folder of your project. The [build output folder](/pages/configuration/build-configuration/) is project-specific, so the `_redirects` file should not always be in the root directory of the repository. Changes to redirects will be updated to your website at build time so make sure you commit and push the file to trigger a new build each time you update redirects.

{{<Aside type="warning">}}

Redirects defined in the `_redirects` file are not applied to requests served by [Functions](/pages/functions/), even if the Function route matches the URL pattern. If your Pages application uses Functions, you must migrate any behaviors from the `_redirects` file to the code in the appropriate `/functions` route, or [exclude the route from Functions](/pages/functions/routing/#create-a-_routesjson-file).

{{</Aside>}}

## Structure

### Per line

Only one redirect can be defined per line and must follow this format, otherwise it will be ignored.

```txt
[source] [destination] [code?]
```

{{<definitions>}}

- `source` {{<prop-meta>}}required{{</prop-meta>}}
  
  - A file path.
  - Can include [wildcards (`*`)](#splats) and [placeholders](#placeholders).
  - Because fragments are evaluated by your browser and not Cloudflare's network, any fragments in the source are not evaluated.
    
- `destination` {{<prop-meta>}}required{{</prop-meta>}}
  
  - A file path or external link. 
  - Can include fragments, query strings, [splats](#splats), and [placeholders](#placeholders).
    
- `code` {{<prop-meta>}}default: `302`{{</prop-meta>}}
  
  - Optional parameter

{{</definitions>}}

Lines starting with a `#` will be treated as comments.

### Per file

A project is limited to 2,000 static redirects and 100 dynamic redirects, for a combined total of 2,100 redirects. Each redirect declaration has a 1,000-character limit.

In your `_redirects` file:

- The order of your redirects matter. If there are multiple redirects for the same `source` path, the topmost redirect is applied.
- Static redirects should appear before dynamic redirects.
- Redirects are always followed, regardless of whether or not an asset matches the incoming request.

A complete example with multiple redirects may look like the following:

```txt
---
filename: _redirects
---
/home301 / 301
/home302 / 302
/querystrings /?query=string 301
/twitch https://twitch.tv
/trailing /trailing/ 301
/notrailing/ /nottrailing 301
/page/ /page2/#fragment 301
/blog/* https://blog.my.domain/:splat
/products/:code/:name /products?code=:code&name=:name
```

{{<Aside type= "note">}}

In the case of some frameworks, such as Jekyll, you may need to manually copy and paste your `_redirects` file to the build output directory. To do this:

1. Log in to the [Cloudflare dashboard](https://dash.cloudflare.com) and select your account.
2. Go to **Workers & Pages** > your Pages project > **Settings** > **Builds & deployments**.
3. Go to **Build configurations** > **Edit configurations** > change the build command to `jekyll build && cp _redirects _site/_redirects` and select **Save**.

{{</Aside>}}

## Advanced redirects

Cloudflare currently offers limited support for advanced redirects. More support will be added in the future.

{{<table-wrap>}}

| Feature                             | Support | Example                                                         | Notes                                   |
| ----------------------------------- | ------- | --------------------------------------------------------------- | --------------------------------------- |
| Redirects (301, 302, 303, 307, 308) | Yes     | `/home / 301`                                                   | 302 is used as the default status code. |
| Rewrites (other status codes)       | No      | `/blog/* /blog/404.html 404`                                    |                                         |
| Splats                              | Yes     | `/blog/* /posts/:splat`                                         | Refer to [Splats](#splats).             |
| Placeholders                        | Yes     | `/blog/:year/:month/:date/:slug /news/:year/:month/:date/:slug` | Refer to [Placeholders](#placeholders). |
| Query Parameters                    | No      | `/shop id=:id /blog/:id 301`                                    |                                         |
| Proxying                            | Yes     | `/blog/* /news/:splat 200`                                      | Refer to [Proxying](#proxying).         |
| Domain-level redirects              | No      | `workers.example.com/* workers.example.com/blog/:splat 301`     |                                         |
| Redirect by country or language     | No      | `/ /us 302 Country=us`                                          |                                         |
| Redirect by cookie                  | No      | `/\* /preview/:splat 302 Cookie=preview`                        |                                         |

{{</table-wrap>}}

## Redirects and header matching

Redirects execute before headers, so in the case of a request matching rules in both files, the redirect will win out.

### Splats

On matching, a splat (asterisk, `*`) will greedily match all characters. You may only include a single splat in the URL.

The matched value can be used in the redirect location with `:splat`.

### Placeholders

{{<render file="_headers_redirects_placeholders.md" withParameters="redirect">}}

```txt
---
filename: _redirects
---
/movies/:title /media/:title
```

### Proxying

Proxying will only support relative URLs on your site. You cannot proxy external domains.

Only the first redirect in your will apply. For example, in the following example, a request to `/a` will render `/b`, and a request to `/b` will render `/c`, but `/a` will not render `/c`.

```
/a /b 200
/b /c 200
```

{{<Aside heading="Proxying's impact on SEO">}}
Be aware that proxying pages can have an adverse effect on search engine optimization (SEO). Search engines often penalize websites that serve duplicate content. Consider adding a `Link` HTTP header which informs search engines of the canonical source of content.

For example, if you have added `/about/faq/* /about/faqs 200` to your `_redirects` file, you may want to add the following to your `_headers` file:

```txt
/about/faq/*
  Link: </about/faqs>; rel="canonical"
```

{{</Aside>}}

## Surpass `_redirects` limits

A [`_redirects`](/pages/platform/limits/#redirects) file has a maximum of 2,000 static redirects and 100 dynamic redirects, for a combined total of 2,100 redirects. Use [Bulk Redirects](/rules/url-forwarding/bulk-redirects/) to handle redirects that surpasses the 2,100 redirect rules limit set by Pages.

{{<Aside type="note">}}

The redirects defined in the `_redirects` file of your build folder can work together with your Bulk Redirects. In case of duplicates, Bulk Redirects will run in front of your Pages project, where your other redirects live.

For example, if you have Bulk Redirects set up to direct `abc.com` to `xyz.com` but also have `_redirects` set up to direct `xyz.com` to `foo.com`, a request for `abc.com` will eventually redirect to `foo.com`.

{{</Aside>}}

To use Bulk Redirects, refer to the [Bulk Redirects dashboard documentation](/rules/url-forwarding/bulk-redirects/create-dashboard/) or the [Bulk Redirects API documentation](/rules/url-forwarding/bulk-redirects/create-api/).

## Related resources

- [Transform Rules](/rules/transform/)
