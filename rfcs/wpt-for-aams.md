# RFC 204: WPT testing for AAMs

## Summary

Extend WPT with the ability to test Accessibility APIs exposed by browsers.

The Accessibility API Mapping (AAM) specifications
([Core-AAM](https://www.w3.org/TR/core-aam-1.2/),
[HTML-AAM](https://www.w3.org/TR/html-aam-1.0/),
[SVG-AAM](https://www.w3.org/TR/svg-aam-1.0/), potentially
[MathML-AAM](https://w3c.github.io/mathml-aam/),
[CSS-AAM](https://w3c.github.io/css-aam/)) describe how user agents
should expose semantics of web content languages to accessibility
APIs. These documents, taken together, describe how the browser should
build an accessibility tree for each web page, and how the tree should
be mapped to various accessibility platform APIs.

## Details

<details>
<summary><h3>Background: Computed Accessibility Tree, Platform Accessibility APIs and the AAMs</h3></summary>

#### The Accessibility Tree and Accessibility APIs
![Diagram showing the interaction between a browser and accessibility APIs. See text below for more detail.](assets/accessibility_apis.png)

<details id="accessibility_apis_diagram">
<summary>Full image description</summary>
A block diagram.

At the top level it shows an assistive technology application, such as
a screen reader using speech or braille, communicating via Platform
Accessibility APIs with a Browser application.

Within the Browser application, it shows that the Assistive Technology
is communicating via a Platform API mapping which is an adapter for
the computed accessibility tree data structure.  In turn, the computed
accessibility tree is built based on other data structures: the DOM
tree and the layout/box tree; the layout/box tree is based on the DOM
tree and CSS.

The computed accessibility tree falls in the center of the diagram, as
it's the source of truth for the platform API mappings which are
queried by assistive technology, and also the result of a complex
process of transformation based on several other data structures.

At the bottom of the diagram is a legend explaining how to read the
different types of arrows and boxes used in the diagram.
</details>

Assistive technologies (ATs) query the browser via platform
accessibility APIs. These are operating system-specific APIs which
provide detailed information about an application's UI state for the
purpose of augmenting the user experience for users with disabilities
which affect their ability to perceive or operate the standard UI.

One example of a commonly used AT is a screen reader, which assists
users with vision impairments by providing a spoken version of the UI.
Typically, a screen reader user will navigate through the UI by
traversing the page and "visiting" UI elements, hearing a spoken
description of each such as "Overview, heading level 2" or "Core-AAM,
link".

The accessibility APIs allow this by making extensive data available
about the UI, in the form of a tree structure referred to as the
"accessibility tree".  The accessibility tree provides information
about each UI element (often corresponding to a DOM node), allowing
screen readers to provide these spoken descriptions.  Each platform's
accessibility APIs provide methods to query the tree structure, and
the information available for each node.

They also typically provide ways to interact with the UI, such as
sending a "click" action to the currently visited UI element, or
selecting a particular option from a picker.  Also, they provide ways
for the UI to provide real-time updates or alerts in case of UI
changing in a way that should be brought to the user's attention.

#### The computed accessibility tree

Since browser engines are typically platform-independent, typically they will compute a generic accessibility tree which can be adapted to any platform's accessibility APIs.
The platform APIs are typically supported using an adapter which maps the platform API on to data from the computed accessibility tree.

#### AAM specifications

The AAM specifications provide mappings between web technologies such as ARIA and HTML, and platform accessibility APIs.

For example, here is the mapping from Core-AAM for an Element with [`aria-checked=true`](https://w3c.github.io/core-aam/#ariaCheckedTrue)

<table>
  <tbody>
    <tr>
      <th><abbr title="Microsoft Active Accessibility">MSAA</abbr> + IAccessible2 </th>
      <td>
        <span>State: <code>STATE_SYSTEM_CHECKED</code></span><br>
        <span>Object Attribute: <code>checkable:true</code></span>
      </td>
    </tr>
    <tr>
      <th><abbr title="User Interface Automation">UIA</abbr></th>
      <td>
        <span>Property: <code>Toggle.ToggleState</code>: <code>On (1)</code></span><br>
        <span>Property: <code>SelectionItem.IsSelected</code>: <code>True</code> for <code>radio</code> and <code>menuitemradio</code></span>
      </td>
    </tr>
    <tr>
      <th><abbr title="Accessibility Toolkit">ATK</abbr>/<abbr title="Assistive Technology-Service Provider Interface">AT-SPI</abbr></th>
      <td>
        <span>State: <code>STATE_CHECKABLE</code></span><br>
        <span>State: <code>STATE_CHECKED</code></span>
      </td>
    </tr>
    <tr>
      <th><abbr title="macOS Accessibility Protocol">AX API</abbr></th>
      <td>
        <span>Property: <code>AXValue</code>: <code>1</code></span><br>
        <span>Property: <code>AXMenuItemMarkChar</code>: <code>✓</code> for <code>menuitemcheckbox</code> and <code>menuitemradio</code></span>
      </td>
    </tr>
  </tbody>
</table>

And here is the mapping from HTML-AAM for a [`<button>`](https://www.w3.org/TR/html-aam-1.0/#el-button):

<table>
  <tbody>
    <tr>
      <th><abbr title="HyperText Markup Language">HTML</abbr> Specification</th>
      <td>
        <a data-type="element" href="https://html.spec.whatwg.org/multipage/form-elements.html#the-button-element"><code>button</code></a>
      </td>
    </tr>
    <tr>
      <th>[<cite><a class="bibref" data-link-type="biblio" href="#bib-wai-aria-1.2" title="Accessible Rich Internet Applications (WAI-ARIA) 1.2">wai-aria-1.2</a></cite>]</th>
      <td><a class="core-mapping" href="https://www.w3.org/TR/core-aam-1.2/#role-map-button"><code>button</code></a> role</td>
    </tr>
    <tr>
      <th><a href="https://www.w3.org/TR/core-aam-1.2/#roleMappingComputedRole">Computed Role</a></th>
      <td class="role-computed"><div class="general">Use <abbr title="Accessible Rich Internet Applications">WAI-ARIA</abbr> mapping</div></td>
    </tr>
    <tr>
      <th>
        <a href="https://msdn.microsoft.com/en-us/library/dd373608%28v=VS.85%29.aspx"><abbr title="Microsoft Active Accessibility">MSAA</abbr></a> + <a href="http://accessibility.linuxfoundation.org/a11yspecs/ia2/docs/html/">IAccessible2</a>
      </th>
      <td>
        <div class="general">Use <abbr title="Accessible Rich Internet Applications">WAI-ARIA</abbr> mapping</div>
      </td>
    </tr>
    <tr>
      <th><a href="https://msdn.microsoft.com/en-us/library/ms726297%28v=VS.85%29.aspx">UIA</a></th>
      <td>
        <div class="general">Use <abbr title="Accessible Rich Internet Applications">WAI-ARIA</abbr> mapping</div>
      </td>
    </tr>
    <tr>
      <th><a href="https://gnome.pages.gitlab.gnome.org/atk/">ATK</a></th>
      <td>
        <div class="general">Use <abbr title="Accessible Rich Internet Applications">WAI-ARIA</abbr> mapping</div>
      </td>
    </tr>
    <tr>
      <th><a href="https://developer.apple.com/reference/appkit/nsaccessibility">AX</a></th>
      <td>
        <div class="general">Use <abbr title="Accessible Rich Internet Applications">WAI-ARIA</abbr> mapping</div>
      </td>
    </tr>
    <tr>
      <th>Comments</th>
      <td>
        A <code>button</code>'s mapping will change if the
        <a class="core-mapping" href="https://www.w3.org/TR/core-aam-1.2/#role-map-button-pressed"><code>aria-pressed</code></a> or <a class="core-mapping" href="https://www.w3.org/TR/core-aam-1.2/#role-map-button-haspopup"><code>aria-haspopup</code></a> attributes are specified.
      </td>
    </tr>
  </tbody>
</table>

</details>

<details>
<summary><h3>Prior art: Get Computed Label, Get Computed Role, potential WebDriver extensions</h3></summary>

![Diagram showing how WebDriver supports getting the Computed Label and Computed Role for an element. Full description below.](assets/webdriver_accessibility_testing.png)

<details id="webdriver_accessibility_diagram">
<summary>Full image description</summary>
A block diagram.

This diagram extends the <a
href="#user-content-accessibility_apis_diagram">diagram in the
"accessibility tree and accessibility APIs" section</a>, showing how
the WPT application can query the browser's Webdriver API
implementation to request the computed role and computed label for an
element.

It adds a block for the WPT application to the left of the Browser application,
below the assistive technology application block,
and a block within the Browser application for the WebDriver API implementation.

Where the assistive technology application queries the Platform API Mapping
via platform accessibility APIs,
the WPT application queries the WebDriver API implementation via HTTP.

Like the Platform API mapping adapter,
the WebDriver API implementation also queries the computed accessibility tree.
</details>

WebDriver and testharness.js support getting the computed
accessibility
[label](https://www.w3.org/TR/webdriver2/#get-computed-label) and
[role](https://www.w3.org/TR/webdriver2/#get-computed-role) for a
particular element.  This allows testing the two core properties for
nodes in the computed, platform-independent accessibility tree,
verifying that the first step of the process for supporting the
platform accessibility APIs has been successfully completed for those
properties.

Computing the platform-independent role is supported in the AAMs,
particularly in HTML-AAM, by the addition to each mapping table of a
"computed" row in addition to the platform-specific API rows. In the
`<button>` example above, in fact, every API and the "Computed"
mapping all defer to the WAI-ARIA mapping for the `"button"`
role. When no equivalent ARIA role exists, the HTML element name with
a prefix of `html-` is used, as in the mapping for
[`<canvas>`](https://www.w3.org/TR/html-aam-1.0/#el-canvas).

Name computation, meanwhile, is complex and important enough to get a
whole spec to itself: [Accessible Name and Description
Computation](https://www.w3.org/TR/accname-1.2/) - as well as
[detailed steps for certain HTML
elements](https://www.w3.org/TR/html-aam-1.0/#accname-computation).

#### Potential WebDriver extensions for accessibility

There is [a proposal](https://github.com/WICG/aom/issues/203) to add an extension to WebDriver
to allow accessing a full "computed accessibility node" for an element.

This would allow fetching a much more extensive set of computed accessibility properties for an element via WebDriver.
Like Computed Label and Computed Role, these would be querying the platform-independent, computed accessibility tree.

</details>

### Proposal: testing AAMs via platform accessibility APIs directly

![Diagram showing the proposed addition: querying the browser from WPT via the platform accessibility APIs. Full description below.](assets/platform_accessibility_api_testing.png)

<details>
<summary>Full image description</summary>
A block diagram.

This diagram extends the <a
href="#user-content-webdriver_accessibility_diagram">diagram in the
"Prior art" section</a>.

It adds a dotted line arrow from the WPT application to the Platform
API mapping adapter within the Browser application, indicating the
addition proposed here to use the Platform Accessibility APIs directly
to test the browser's support for those APIs.

Notably, while the existing mechanism for WPT to query the browser via
WebDriver communicates with the WebDriver API implementation within
the browser via HTTP, the proposed addition bypasses WebDriver to
query the Platform API Mapping directly via Platform Accessibility
APIs.  This matches the way that Assistive Technology communicates
with the browser application.  </details>

As described in the "Prior Art" section above, we think testing the
computed accessibility tree is extremely useful in itself, and a good
fit for the platform-independent nature of browser testing and
WebDriver specifically.  It provides a good foundation for ensuring
that the first stage of accessibility API mapping for web features has
been implemented, since in almost every case browsers will need to
implement the cross-platform tree as a prerequisite for implementing
the platform-specific APIs.

This proposal focuses on an equally important goal, directly testing
that each browser's platform API support conforms to the AAM
specifications. The easiest and most reliable way to test that is to
use the APIs directly, matching the way that assistive technology
communicates with the browser.

An alternative would be to extend WebDriver to mirror each platform API's vocabulary.
This would involve a great deal of work to specify how those vocabularies should be expressed
through WebDriver and testdriver,
and implementation work to implement those APIs and test the implementation,
before we could even write any tests using the WebDriver version.

<details>
<summary><h3>Example: computed role vs. platform API mapping for <code>&lt;input type="password"&gt;</code>
</h3>
</summary>

The table below is excerpted from the <a href="https://www.w3.org/TR/html-aam-1.0/#el-input-password" id="el-input-password">HTML-AAM mapping for `input` (`type` attribute in the Password state)</a>:

<table aria-labelledby="el-input-password">
  <tbody>
    <tr>
      <th>[<cite><a class="bibref" data-link-type="biblio" href="#bib-wai-aria-1.2" title="Accessible Rich Internet Applications (WAI-ARIA) 1.2">wai-aria-1.2</a></cite>]</th>
      <td>No corresponding role</td>
    </tr>
    <tr>
      <th><a href="https://www.w3.org/TR/core-aam-1.2/#roleMappingComputedRole">Computed Role</a></th>
      <td class="role-computed"><div class="general">html-input-password</div></td>
    </tr>
    <tr>
      <th>
        <a href="https://msdn.microsoft.com/en-us/library/dd373608%28v=VS.85%29.aspx"><abbr title="Microsoft Active Accessibility">MSAA</abbr></a> + <a href="http://accessibility.linuxfoundation.org/a11yspecs/ia2/docs/html/">IAccessible2</a>
      </th>
      <td>
        <div class="role"><span class="type">Role:</span> <code>ROLE_SYSTEM_TEXT</code></div>
        <div class="states"><span class="type">States:</span> <code>STATE_SYSTEM_PROTECTED</code>; <code>IA2_STATE_SINGLE_LINE</code>; <code>STATE_SYSTEM_READONLY</code> if readonly, otherwise <code>IA2_STATE_EDITABLE</code></div>
      </td>
    </tr>
    <tr>
      <th><a href="https://msdn.microsoft.com/en-us/library/ms726297%28v=VS.85%29.aspx">UIA</a></th>
      <td>
        <div class="ctrltype"><span class="type">Control Type:</span> <code>Edit</code></div>
        <div class="properties"><span class="type">Localized Control Type:</span> <code>"password"</code></div>
        <div class="properties"><span class="type">Other properties: </span>Set <code>isPassword</code> to <code>true</code></div>
      </td>
    </tr>
    <tr>
      <th><a href="https://gnome.pages.gitlab.gnome.org/atk/">ATK</a></th>
      <td>
        <div class="role"><span class="type">Role:</span> <code>ATK_ROLE_PASSWORD_TEXT</code></div>
        <div class="states"><span class="type">States:</span> <code>ATK_STATE_SINGLE_LINE</code>; <code>ATK_STATE_READ_ONLY</code> if readonly, otherwise <code>ATK_STATE_EDITABLE</code></div>
      </td>
    </tr>
    <tr>
      <th><a href="https://developer.apple.com/reference/appkit/nsaccessibility">AX</a></th>
      <td>
        <div class="role"><span class="type">AXRole:</span> <code>AXTextField</code></div>
        <div class="subrole"><span class="type">AXSubrole:</span> <code>AXSecureTextField</code></div>
        <div class="roledesc"><span class="type">AXRoleDescription:</span> <code>"secure text field"</code></div>
      </td>
    </tr>
  </tbody>
</table>

Since each platform accessibility API has its own vocabulary and its own conceptual framework,
the concept of "password field" is expressed differently in each API.

Where the computed accessibility tree on each platform would contain a node
with the role `"html-input-password"`,
the browser would need to adapt that computed role to each platform API correctly
in order for users of the assistive technologies which query those APIs
to be able to use the browser.
</details>

## Potential Solutions

### 1. Extend testharness.js with new API end point used to execute tests on backend

We have an [PR open on WPT to extend
testharness.js](https://github.com/web-platform-tests/wpt/pull/53733)
which adds a new endpoint to test_driver which will take a set of
asserts and execute them against the accessibility APIs in the python
layer.

### 3. Add a new test type `aamtest` inspired by `wdspec`

Similarly, we have an [PR open on WPT for a new test
type](https://github.com/web-platform-tests/wpt/pull/53733) which is
modeled after `wdspec` tests and re-uses some of the same classes and
python test fixture.


<details>
<summary><h3>Open questions on technical implementations and test design</h3>
</summary>

The patch includes all relevant technologies and the basic steps
necessary to test the platform-specific accessibility APIs. However,
there are some open design questions which we would love feedback on.

#### Adding dependencies on the Python bindings for the platform APIs

We use [comtypes](https://pypi.org/project/comtypes/) on Windows,
[PyGObject](https://pygobject.gnome.org/index.html) on Linux, and
[pyobjc-framework-ApplicationServices](https://pypi.org/project/pyobjc-framework-ApplicationServices/)
and
[pyobjc-framework-Accessibility](https://pypi.org/project/pyobjc-framework-Accessibility/)
on macOS.

How do we ensure that WPT users have these libraries available?

#### Using the `testharness` test type rather a new test type.

Either way, we can use webdriver to interact with the browser for more
complicated tests. Each test will be some html, potentially CSS and
Javascript, and queries against the exposed accessibility APIs in
python. We can either make the test an html file that sends commands
to python to execute on the backend, or a python file that load an
HTML page via webdriver.


#### Per-platform tests

Currently, each test tests a "mapping" in the AAM, for example, how a
`button` element is exposed in accessiblity APIs. A single file
corrosponds to the mappings for `button` and there is a subtest for
each accessibility API. If you are testing the mac API on linux, the
mac API passes trivially (no assertions run) but ideally it should
have a test type such as "not applicable".

#### [Getting the browser PID](https://github.com/w3c/webdriver/issues/1823)

In order to query the browser application via accessibility APIs,
we need a reliable way to find the correct browser application.
The [proof of concept](https://github.com/Igalia/wpt/pull/2/files)
uses the application name;
however this creates obvious problems if you're trying to use a particular browser
at the same time as WPT is running a separate instance of the same browser for testing.

So far, the best way we've found is to use the process ID;
this allows us to unambiguously find the browser process under test.
However, since WPT often doesn't start browsers directly,
but uses an automation tool like Marionette or ChromeDriver,
we can't always get the process ID of the browser easily from WPT.

Valerie has filed [an issue](https://github.com/w3c/webdriver/issues/1823)
on WebDriver to propose adding a mechanism to request the browser PID,
potentially through the `capabilities` object used when creating a Session.

</details>

## Risks

These test might be slow to to run. The browser is noticeably slower when accessibility features are turn on (when the browser builds the accessibility tree).
Additionally, the browser can't run in headless mode.

The tests may be flaky due to timing issues with the accessibility tree being built. The accessibility tree lags behind the DOM and even paint, with no feedback available to the page about what the status of the accessibility tree is.
The webdriver computed name/computed role APIs sidestep this issue, since they build the accessiblity tree on demand.

These tests introduce platform specific tests and results, which increase the complexity of WPT.

