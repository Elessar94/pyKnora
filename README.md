# pyKnora
This library consists of
- ```knora.py``` Python modules for accessing Knora using the API (ontology creation, data import/export etc.)
- ```create_ontology.py``` A program to create an ontology out of a simple JSON description

## JSON ontology definition format

The JSON file contains a first object an object with the ```prefixes``` for
external ontologies that are being used, followed by the definition of
the project wic h includes all resources and properties:

### Prefixes

```json
{
  "prefixes": {
    "foaf": "http://xmlns.com/foaf/0.1/",
    "dcterms": "http://purl.org/dc/terms/"
  },
  "project": {…},
  
}
```

### Project data
The project definitions requires

- _"shortcode"_: A hexadecimal string in the range between "0000" and "FFFF" uniquely identifying the project. 
- _"shortname"_: A short name (string)
- a _"longname"_: A longer string giving the full name for the project
- _descriptions_: Strings describing the projects content. These
  descriptions can be supplied in several languages (currently _"en"_, _"de"_, _"fr"_ and _"it"_ are supported).
  The descriptions have to be given as JSON object with the language as key
  and the description as value. At least one description in one language is required.
- _keywords_: An array of keywords describing the project.
- _lists_: The definition of flat or hierarchical list (thesauri, controlled vocabularies)
- _ontology_: The definition of the data model (ontology)

This a project definition lokks like follows:
  
```json
"project": {
   "shortcode": "0809",
   "shortname": "tesex"
   "longname": "Test Example",
   "descriptions": {
     "en": "This is a simple example project with no value.",
     "de": "Dies ist ein einfaches, wertloses Beispielproject"
   }
   "keywords": ["example", "senseless"],
   "lists": […],
   "ontology": {…}
}
```

### Lists
A List consists of a root node identifing the list and an array of subnodes.
Each subnode may contain again subnodes (hierarchical list).
A node has the following elements:

- _name_: Name of the node. Should be unique for the given list
- _labels_: Language dependent labels
- _comments_: language dependent comments (optional)
- _nodes_: Array of subnodes (optional – leave out if there are no subnodes)

The _lists_ object contains an array of lists. Here an example:

```json
    "lists": [
      {
        "name": "orgtpye",
        "labels": { "de": "Organisationsart", "en": "Organization Type" },
        "nodes": [
          {
            "name": "business",
            "labels": { "en": "Commerce", "de": "Handel" },
            "comments": { "en": "no comment", "de": "kein Kommentar" },
            "nodes": [
              {
                "name": "transport",
                "labels": { "en": "Transportation", "de": "Transport" }
              },
              {
                "name": "finances",
                "labels": { "en": "Finances", "de": "Finanzen" }
              }
            ]
          },
          {
            "name": "society",
            "labels": { "en": "Society", "de": "Gesellschaft" }
          }
        ]
      }
    ]
```
the _lists_ element is optional.

## Ontology

The _ontology_ object contains the definition of the data model. The ontology has
the following elemens:

- _name_: The name of the ontology. This has to be a CNAME conformant name that can be use as prefix!
- _label_: Human readable and understandable name of the ontology
- _resources_: Array defining the resources (entities) of the data model

```json
    "ontology": {
      "name": "teimp",
      "label": "Test import ontology",
      "resources": […]
    }
```

### Resources
The resource classes are the primary entities of the data model. A resource class
is a template for the representation of a real object that is represented in
the DaSCh database. A resource class defines properties (aka _data fields_). For each of
these properties a data type as well as the cardinality have to defined.

A resource consists of the following definitions:

- _name_: A name for the resource
- _label_: The string displayed of the resource is being accessed
- _super_: A resource class is always derived from an other resource. The
  most generic resource class Knora offers is _"Resource"_. The following
  parent predefined resources are provided by knora:
  - _Resource_: A generic "thing" that represents an item from the reral world
  - _StillImageRepresentation_: An object that is connected to a still image
  - _TextRepresentation_: An object that is connected to an (external) text (Not Yet Implemented)
  - _AudioRepresentation_: An object representing audio data (Not Yet Implemented)
  - _DDDRepresentation_: An object representing a 3d representation (Not Yet Implemented)
  - _DocumentRepresentation_: An object representing a opaque document (e.g. a PDF)
  - _MovingImageRepresentation_: An object representing a moving image (video, film)
  - _Annotation_: A predefined annotation object. It has the following properties
  defined:
    - _hasComment_ (1-n), _isAnnotationOf_ (1)
  - _LinkObj_: An resource class linking together several other, generic, resource classes. The class
  has the following properties: _hasComment_ (1-n), _hasLinkTo_ (1-n)
  - _Region_: Represents a simple region. The class has the following properties:
  _hasColor_ (1), _isRegionOf_ (1) _hasGeometry_ (1), _isRegionOf_ (1), _hasComment_ (0-n)
  
  However, a resource my be derived from a resource class in another ontology within the same project or
  from another resource class in the same ontology. In this case the reference
  has to have the form _prefix_:_resourceclassname_.
- _labels_: Language dependent, human readable names
- _comments_: Language dependend comments (optional)
- _properties_: Array of property definition for this resource class.

Example:

```json
     "resources": [
        {
          "name": "person",
          "super": "Resource",
          "labels": { "en": "Person", "de": "Person" },
          "comments": {
            "en": "Represents a human being",
            "de": "Repräsentiert eine Person/Menschen"
          },
          "properties": […]
        }
```

#### Properties
Properties are the definition of the data fields a resource class may or must have.
The properties object has the following fields:

- _name_: A name for the property
- _super_: A property has to be derived from at least one base property. The most generic base property
  Knora offers is _hasValue_. In addition the property may by als a subproperty of
  properties defined in external ontologies. In this case the qualified name including
  the prefix has to be given.
  The following base properties are definied by Knora:
  - _hasValue_: This is the most generic base.
  - _hasLinkTo_: This value represents a link to another resource. You have to indicate the
    the "_object_" as a prefixed IRI that identifies the resource class this link points to.
  - _hasColor_: Defines a color value (_ColorValue_)
  - _hasComment_: Defines a "standard" comment
  - _hasGeometry_: Defines a geometry value (a JSON describing a polygon, circle or rectangle), see _ColorValue_
  - _isPartOf_: A special variant of _hasLinkTo_. It says that an instance of the given resource class
    is an integral part of another resource class. E.g. a "page" is a prt of a "book".
  - _isRegionOf_: A special variant of _hasLinkTo_. It means that the given resource class
    is a "region" of another resource class. This is typically used to describe regions
    of interest in images.
  - _isAnnotationOf_: A special variant of _hasLinkTo_.  It denotes the given resource class
    as an annotation to another resource class.
  - _seqnum_: An integer that is used to define a sequence number in an ordered set of
    instances.
  
- _object_: The "object" defines the type of the value that the property will store.
  The following object types are allowed:
  
  - _TextValue_: Represents a text that may contain standoff markup
  - _ColorValue_: A string in the form "#rrggbb" (standard web color format)
  - _DateValue_: represents a date. It is a string having the format "_calendar":"start":"end"
    - _calender_ is either _GREGORIAN_ or _JULIAN_
    - _start_ has the form _yyyy_-_mm_-_dd_. If only the year is given, the precision
      is to the year, of only the year and month are given, the precision is to a month.
    - _end_ is optional if the date represents a clearely defined period or uncertainty.
    
    In total, a DateValue has the following form: "GREGORIAN:1925:1927-03-22"
    which means antime in between 1925 and the 22nd March 1927.
  - _DecimalValue_: a number with decimal point
  - _GeomValue_: Represents a geometrical shape as JSON.
  - _GeonameValue_: Represents a location ID in geonames.org
  - _IntValue_: Represents an integer value
  - _BooleanValue_: Represents a Boolean ("true" or "false)
  - _UriValue_: : Represents an URI
  - _IntervalValue_: Represents a time-interval
  - _ListValue_: Represents a node of a (possibly hierarchical) list
- _labels_: Language dependent, human readable names
- _gui_element_: The gui_element is stricly seen not part of the data. It gives the
  generic GUI a hint about how the property should be presented to the used.
  There are the following gui_elements available:
  - :Colorpicker     -> ncolors=integer
  - :Date
  - :Geometry
  - :Geonames
  - :Interval
  - :List            -> hlist(required)=<iri>
  - :Pulldown        -> hlist(required)=<iri>
  - :Radio           -> hlist(required)=<iri>
  - :Richtext
  - :Searchbox       -> numprops=integer
  - :SimpleText      -> maxlength=integer, size=integer
  - :Slider          -> max(required)=decimal, min(required)=decimal
  - :Spinbox         -> max=decimal, min=decimal
  - :Textarea        -> cols=integer, rows=integer, width=percent, wrap=string(soft|hard)
  - :Checkbox
  - :Fileupload

- _gui_attributes_:
- _cardinality_:
