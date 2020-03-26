# Traceability Tooling

This proposal is intended to support and help inform the [Traceability
Proposal](./traceability.md) for Informal Systems.

## Background

### Requirements Management

[Requirements management][1] is a broad and deep area of interest in software
engineering. This breadth and depth is most clearly evident in safety-critical
software engineering efforts, such as in aerospace and military application
development, where mistakes can unnecessarily cost lives.

Notable examples of systems engineering standardization efforts that include
requirements management:

* [ISO/IEC 12207][2], which replaced [MIL-STD-498][3] for military software
  engineering.
* [DO-178C][4] for aerospace systems engineering, which embraces the use of
  formal methods for verification.

### Existing Technologies

Many technologies already exist to assist in requirements management. Some
prominent examples include:

* [IBM Engineering Requirements Management DOORS][6]
  * Video: [IBM Rational DOORS Objects][7]
  * Video: [IBM Rational DOORS Attributes][8]
  * Video: [IBM Rational DOORS Linking and Traceability][9]
  * [Product Documentation][10]
* [Eclipse Capra][17]
  * [Traceability demonstration][20]
  * [Video demonstration][18]
* [Modern Requirements][11]
* [Pearls][15]
* [Visure Requirements][16]
* [Jama Connect][13]

Additional potentially useful/interesting links:

* [Requirements Interchange Format][19]
* See the video series [Using Qualified Tools in a DO-178C Development
  Process][5].

### Domain Analysis

When looking at the existing software packages available for requirements
management, it seems, at a high level, that the following primary concepts are
employed:

* **Specifications**, which:
  * contain requirements,
  * have attributes/metadata, and
  * have relationships to other specifications, where these relationships are
    directional and can have other arbitrary attributes associated with them.
* **Requirements**, which have:
  * globally unique identifiers,
  * relationships to other requirements, where these relationships are
    directional and can have other arbitrary attributes associated with them,
  * other arbitrary attributes, such as priority and status (e.g. "Accepted",
    "Rejected").
* **Implementation artifacts**, such as source code, which implement
  requirements.
* **Acceptance criteria**, which:
  * have globally unique identifiers,
  * relate to specific requirements, and
  * specify what is necessary to consider a requirement as being met.
* **Test artifacts**, which, when executed successfully, automatically verify
  that specific acceptance criteria are met.

### Versioning

It seems as though most requirements management tooling implements their own
complex versioning system on top of their domain model. If, however, one had to
devise a scheme that allows people to specify the domain objects in plain text
such that it could be stored in a VCS like Git, some aspects of this versioning
would be available for free.

### Visualization

Most requirements management tools provide various ways of visualizing
requirements:

* [Traceability matrices][21]
* Traceability graphs
* Hierarchically structured documents (e.g. Microsoft Word documents with
  different levels of requirements specified at different heading levels and
  paragraph text)

## Desired User Experience

## Implementation Recommendations

[1]: https://en.wikipedia.org/wiki/Requirements_management
[2]: https://en.wikipedia.org/wiki/ISO/IEC_12207
[3]: https://en.wikipedia.org/wiki/MIL-STD-498
[4]: https://en.wikipedia.org/wiki/DO-178C
[5]: https://www.mathworks.com/videos/series/using-qualified-tools-in-a-do-178c-development-process.html
[6]: https://www.ibm.com/us-en/marketplace/requirements-management
[7]: https://www.youtube.com/watch?v=GAY9Xq1dcsU&list=PLFB5C518530CFEC93&index=1
[8]: https://www.youtube.com/watch?v=zfp5DEisdcs&list=PLFB5C518530CFEC93&index=2
[9]: https://www.youtube.com/watch?v=2tN_cVQP214&list=PLFB5C518530CFEC93&index=4
[10]: https://www.ibm.com/support/knowledgecenter/SSYQBZ_9.7.1/com.ibm.doors.requirements.doc/helpindex_doors.html
[11]: https://www.modernrequirements.com/
[12]: https://reqtest.com/
[13]: https://www.jamasoftware.com/platform/jama-connect/
[14]: https://www.xebrio.com/requirements-management-software
[15]: https://pearls-inc.com/products/pearls-lite-version/
[16]: https://visuresolutions.com/requirements-management-tool
[17]: https://projects.eclipse.org/projects/modeling.capra
[18]: https://www.youtube.com/watch?v=h6qntRT33gM
[19]: https://www.omg.org/spec/ReqIF/
[20]: https://www.youtube.com/watch?v=XRtLs5OT_yM
[21]: https://en.wikipedia.org/wiki/Traceability_matrix

