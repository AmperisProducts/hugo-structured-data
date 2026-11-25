# hugo-structured-data

A [Hugo](https://gohugo.io/) module that renders [schema.org](https://schema.org/) structured data as one JSON-LD `@graph` per page, for multilingual sites.

Each page gets the nodes that apply to it, linked by `@id`:

Node | When
:-- | :--
`Organization` | When you configure an organization. One `@id` shared by every language.
`WebSite` | Always, one per language, with `inLanguage`.
`WebPage` or `CollectionPage` | Always. Includes the description, dates, main image and breadcrumb.
`BreadcrumbList` | On every page except the home page, from the page's ancestors.
`Article` (or `BlogPosting`, `NewsArticle`, `TechArticle`) | On regular pages of the sections you choose.
`Product` | When the front matter asks for it, with brand, SKU, MPN, GTIN, category and an optional offer.
`FAQPage` | When the front matter has a `faq` list.

Every node is built as a Hugo map and serialized with `jsonify`, so the output is always valid JSON, with correct escaping, whatever your content contains. Dates use ISO 8601, URLs are absolute, and `inLanguage` uses the language locale.

## Requirements

Hugo 0.158.0 or later. The standard edition is enough.

## Installation

Import the module in your project configuration:

```toml
[module]
  [[module.imports]]
    path = "github.com/AmperisProducts/hugo-structured-data"
```

Your project must be a Hugo module (`hugo mod init github.com/you/your-site`), and Go must be installed to fetch the module. See [Use Hugo Modules](https://gohugo.io/hugo-modules/use-modules/).

Then call the partial in the `head` element of your base template:

```go-html-template
{{ partial "structured-data/jsonld.html" . }}
```

## Configuration

```toml
[params.structuredData]
  articleSections = ["blog"]   # empty means the site's main sections
  articleType = "BlogPosting"  # Article, BlogPosting, NewsArticle or TechArticle
  breadcrumbs = true

  [params.structuredData.organization]
    type = "Organization"      # or a subtype, such as LocalBusiness
    name = "Example Ltd"
    legalName = "Example Limited"
    url = "https://example.org/"
    logo = "/images/logo.png"
    email = "info@example.org"
    telephone = "+34 000 000 000"
    vatID = "ESB00000000"
    sameAs = ["https://www.linkedin.com/company/example"]
    [params.structuredData.organization.address]
      streetAddress = "Main Street 1"
      addressLocality = "Lugo"
      postalCode = "27003"
      addressCountry = "ES"
```

Empty values are left out of the output.

## Front matter

Disable the structured data of one page:

```yaml
structuredData: false
```

Describe a product:

```yaml
title: Insulation tester X1
description: A 5 kV insulation resistance tester.
images: [/images/x1.jpg]
structuredData:
  type: Product
  product:
    brand: Acme
    manufacturer: Acme Instruments
    sku: X1-5KV
    mpn: 100-200
    category: Insulation testers
    offers:            # optional; leave it out if you sell on quotation
      price: 1250
      priceCurrency: EUR
      availability: InStock
```

Add questions and answers (the answers are rendered as Markdown):

```yaml
faq:
  - question: What is the maximum test voltage?
    answer: "**5 kV** DC."
```

Force a type on any page with `structuredData.type`: `Article`, `BlogPosting`, `NewsArticle`, `TechArticle`, `Product`, or `WebPage` for none of them.

### Images

The module uses, in this order: the `images` front matter field (the field Hugo's embedded Open Graph templates use), then a page resource whose name contains `feature`, `cover` or `thumbnail`, then the first image resource of the page.

### Authors

Articles use the `author` front matter field, a name or a list of names. Without it, the organization is the author.

## Validate

Check a few pages with the [Schema Markup Validator](https://validator.schema.org/) and Google's [Rich Results Test](https://search.google.com/test/rich-results). Google shows product rich results only for products with offers, reviews or ratings.

The `exampleSite` directory contains a multilingual example. To build it: `cd exampleSite && hugo build`.

## License

[Apache License 2.0](LICENSE). Made by [Amperis Products S.L.](https://github.com/AmperisProducts), which uses this approach on its own multilingual Hugo site.
