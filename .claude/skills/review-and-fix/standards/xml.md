# XML Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers XML documents, schemas (XSD), XSLT, XML config files, and **code in any language that parses or builds XML**.

## Authorities
- W3C XML 1.0, Namespaces in XML, XML Schema 1.1, XSLT 3.0 and XPath 3.1
- OWASP XML External Entity Prevention Cheat Sheet
- CWE-611 (XXE) and CWE-776 (entity expansion, "billion laughs")

## Tooling to look for and run
- `xmllint --noout file.xml` for well-formedness
- `xmllint --noout --schema schema.xsd file.xml` for validity
- Project schema validation in the build
- `mvn validate` for POMs

## Parser hardening (XXE) — `[C]` when parsing untrusted XML
| Platform | Safe configuration |
|---|---|
| Java `DocumentBuilderFactory` / `SAXParserFactory` | `setFeature("http://apache.org/xml/features/disallow-doctype-decl", true)`; also `setXIncludeAware(false)`, `setExpandEntityReferences(false)` |
| Java `XMLInputFactory` (StAX) | `setProperty(XMLInputFactory.SUPPORT_DTD, false)` and `IS_SUPPORTING_EXTERNAL_ENTITIES` set to false |
| Java `TransformerFactory` / `SchemaFactory` | `setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "")` and `ACCESS_EXTERNAL_STYLESHEET` / `ACCESS_EXTERNAL_SCHEMA` set to `""` |
| .NET `XmlReader` | `DtdProcessing = DtdProcessing.Prohibit`, `XmlResolver = null` (`XmlDocument` and `XmlTextReader` on .NET Framework before 4.5.2 are unsafe by default) |
| Python | `defusedxml` instead of `xml.etree`, `minidom`, `sax` or `lxml` for untrusted input; for `lxml`, `resolve_entities=False, no_network=True` |
| PHP | `libxml_disable_entity_loader` (PHP < 8) and never pass `LIBXML_NOENT` |
| Node | Avoid `noent: true` in libxmljs; prefer parsers with no DTD support |

## Security
- `[C]` XSLT loaded from untrusted sources, or with script or extension functions enabled. Examples: `XsltSettings.EnableScript = true` in .NET; Xalan extensions with `FEATURE_SECURE_PROCESSING` turned off in Java.
- `[C]` Secrets in committed config XML: connection-string passwords in `web.config`, `context.xml` or `persistence.xml`; Maven `settings.xml` credentials; API keys.
- `[H]` XPath built by concatenating user input (XPath injection). Use parameterized XPath (variable resolvers).
- `[H]` XML built by string concatenation with untrusted values (breaks escaping, injects elements). Use a DOM or StAX writer, or a serializer.
- `[H]` `web.config`: `<compilation debug="true">`, `<customErrors mode="Off">`, or `<httpCookies>` without `requireSSL` / `httpOnlyCookies` in production transforms.
- `[H]` `AndroidManifest.xml`: `android:debuggable="true"`; `android:exported="true"` on components without permission checks; `usesCleartextTraffic="true"`; `allowBackup="true"` for apps with sensitive data. See [android.md](android.md) for the full Android checklist.
- `[M]` Spring XML or other DI config exposing actuator or admin beans; permissive security interceptors.

## Build files
**`pom.xml`:**
- `[H]` Repositories over `http://`. Maven 3.8.1 and later blocks them.
- `[H]` Dependency versions with known CVEs.
- `[H]` SNAPSHOT dependencies in a release.
- `[M]` Versions hardcoded per module instead of in `<dependencyManagement>` or the BOM.
- `[M]` Duplicate dependencies; test-only libraries without `<scope>test</scope>`; plugins without versions.

**`.csproj` / `Directory.Packages.props`:**
- `[H]` Vulnerable `PackageReference` versions.
- `[M]` Floating versions (`*`) where the repo uses central package management.

## Correctness
- `[H]` Not well-formed: unescaped `&` or `<`, mismatched tags, multiple root elements.
- `[H]` Not valid against the XSD the system relies on.
- `[H]` Namespace mistakes. A default namespace makes un-prefixed XPath like `/root/item` match nothing. Register a prefix.
- `[M]` The encoding declaration doesn't match the actual bytes, or there's a BOM before `<?xml`. `xs:dateTime` or `xs:decimal` values in a locale-specific format.
- `[M]` Whitespace treated as insignificant in mixed content where it matters; CDATA used to dodge escaping of data that can itself contain `]]>`.

## Maintainability
- `[M]` A new element or attribute added to instances but not to the schema; schema changes that aren't backward compatible with existing documents (needs-decision).
- `[L]` Inconsistent indentation or naming (camelCase vs kebab-case) compared with the rest of the document; commented-out blocks left in config.

## Appian exports
Exported Appian application XML is generated. Review its SAIL with [appian.md](appian.md), but don't hand-edit UUIDs, versions or references.
