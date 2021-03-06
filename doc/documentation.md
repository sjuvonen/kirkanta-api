Kirkanta API v4
===============

Date        | API version       | Summary of changes
----------- | ----------------- | ------------------
2018-08-27  | 4.0.0-preview     | Published a preview version.
2018-09-26  | 4.0.0-preview     | Added `links` block on `/finna_organisation`.
2018-10-12  | 4.0.0-preview     | Enabled HTML in service.description.
2018-10-17  | 4.0.0-preview     | New sections for libraries: `emailAddresses` and `links`.
2018-10-17  | 4.0.0-preview     | Image entries now contain resolution and filesize information.
2018-10-17  | 4.0.0-preview     | Added `library.coverPhoto`.
2018-11-29  | 4.0.0-beta        | Dropped option for returning all translations in a single query.
2018-12-17  | 4.0.0-beta        | Calculate distance to libraries in kilometers instead of meters.
2018-12-17  | 4.0.0-beta        | Remove type `main_library` and rename `library` to `municipal`.

**Documentation for old API versions (in Finnish)**:
[API V3](https://api.kirjastot.fi/v3-doc.html),
[API V2](https://api.kirjastot.fi/v2-doc.html),
[API V1](https://api.kirjastot.fi/v1-doc.html)

# Introduction
Kirjastot.fi offers a free and public API for accessing data of all Finnish libraries. The library
database consists of municipal libraries as well as mobile libraries and faculty libraries of
universities and other educational organisations.

Report technical errors: tekniikka@kirjastot.fi
Report errors in content: toimitus@kirjastot.fi

## Terms of Use
The API can be used free of charge and without authentication. There are no usage limits per se,
but we reserve the right to block any clients deemed to be generating excessive amounts of
*dumb queries*.

## Available languages
Content is produced independently by libraries. For the most part content is provided in Finnish,
while availability of English and Swedish translations is regional. Russian and Sami languages
are also present but in a very restricted manner.

# Endpoints
## Standard endpoints

Endpoint        | Description
--------------- | -----------
/library        | Libraries including mobile libraries.
/consortium     | Library consortiums.
/service        | Common service data.
/schedules      | Opening times for libraries and service points.
/city           | Cities.

## Non-library extensions
These extensions have been created to allow for better integration with Finna.fi.

Endpoint            | Description
------------------- | -----------
/service_point      | Combines both libraries and non-library service points.
/finna_organisation | Extension of `/consortium`. Provides additional Finna-specific data.

## Common parameters
### More comprehensible output
Improve readability of generated JSON by using parameter `pretty`.

### Pagination
By default every query returns at most 10 records. Pagination can be controlled with parameters
`limit` for result size and `skip` for starting from nth record.

### Content language
Identifier  | Language
----------- | --------
fi          | Finnish
en          | English
sv          | Swedish
ru          | Russian
se          | Sami language

Unlike in previous versions of the API, results contain only those languages that data is actually
provided for. Every translatable field inside a record will contain the same set of language fields,
but from one record to another the set of fields might vary.

By default queries return data for all available languages. To fetch only one language at a time,
it is possible to use parameter `lang`.

### Inclusion of related data
Queries allow fetching some relations within the same query. When parameter `refs` is applied, the
result set will contain an additional property `references` that contains a map of each relation
keyed by the type of the sub-document.

#### Example response
```json
https://api.kirjastot.fi/v4/library?id=84834,84925&refs=city

{
    "total": 2,
    "type": "library",
    "result": [{...}, {...}],
    "references": {
        "city": {
            "15863": {
                "id": 15863,
                "name": {
                    "en": "Espoo",
                    "fi": "Espoo",
                    "sv": "Esbo",
                }
            }
        }
    }
}
```

## Rich text in documents
Some fields, e.g. `library.description` and `consortium.description`, contain HTML-formatted text. We
allow use of basic formatting such as `<b>`, `<i>`, `<blockquote>` and lists `<ul>` and `<ol>`. Text is
structured using headings `<h1>` to `<h6>` and paragraphs `<p>`.

# Libraries
```urls
https://api.kirjastot.fi/v4/library
https://api.kirjastot.fi/v4/library/<id>
```

## Query parameters
Name            | Description
--------------- | -----------
id              | List of library IDs.
name            | Name of the library. Varies by `langcode`.
slug            | URL identifier. Varies by `langcode`.
type            | Type of library.
status          | Filter by live status of the library. (see `/schedules`)
--              | --
city            | List of city IDs.
city.name       | Name of the city. Varies by `langcode`.
consortium      | List of consortium IDs.
consortium.name | Name of library consortium.
service         | List of service IDs offered by libraries.
service.name    | Search using a service's name.
--              | --
geo.pos         | Reference point for geographic search. (`lat,lon`)
geo.dist        | Distance from the reference point in kilometers. (`1` = 1 km, `100` = 100 km)
--              | --
created.after   | Lower bound for the date the document was created.
created.before  | Upper bound for the date the document was created.
modified.after  | Lower bound for document modification date.
modified.before | Upper bound for document modification date.
--              | --
with            | Return additional data blocks.
refs            | Collect specified linked data.

- Consortium is applicable only to municipal libraries.
- When using `geo.pos`, calculated distance is returned in field `distance`.
- When using `status`, live status of the library is returned in field `liveStatus`.

## Types of libraries
Identifier          | Description
------------------- | ------
municipal           | Regular municipal library
mobile              | Mobile library (a car)
--                  | --
children            | Children's library
home_service        | A home service library
institutional       | Institutional library
music               | Music library
special             | Special library
--                  | --
polytechnic         | Polytechnic library
school              | Primary school library
university          | University library
vocational_college  | Vocational college library

## Additional data blocks
These blocks can be included into the results using parameter `with`.

Identifier      | Description
--------------- | -----------
mail_address    | Mail address of the library.
phone_numbers   | List of phone numbers.
persons         | List of staff.
pictures        | List of photos.
links           | Links to websites related to this library.
services        | List of services provided by the library.
departments     | List of departments attached to the library.
schedules       | Service times for specified period of time. See endpoint `/schedules`.

- Amount of returned service times can be controlled with parameters `period.start` and `period.end`.
- Note that the maximum number of schedules per request is limited internally to 5 000 rows.

## Related data
Include related data into the results using parameter `refs`.

Identifier      | Description
--------------- | -----------
city            | List of municipalities (based on `library.city`)
consortium      | List of consortiums (based on `library.consortium`)
period          | List of opening time templates (based on `library.schedules.period`)

# Library consortiums
```urls
https://api.kirjastot.fi/v4/consortium
https://api.kirjastot.fi/v4/consortium/<id>
```

Library consortiums join municipal libraries from different municipalities together. Note that
consortiums are defined only for municipal libraries and not other types of libraries.

## Parameters
Name    | Description
------- | -----------
id      | List of consortium IDs.
name    | Consortium name. Varies by `langcode`.
slug    | URL identifier. Varies by `langcode`.

# Service hours
```
https://api.kirjastot.fi/v4/schedules
```

Opening times are provided for libraries and service points. The data consists of days, each of which
contains a list of service times. Days can contain multiple time entries, because libraries can service
either in self-service mode without staff, or while the staff is present as usual.

- Some libraries are closed during the day, resulting in gaps between time entries.
- When a library is closed for the whole day, `times` will be `NULL` and `closed` will be `TRUE`.

## Live status
To return live status for libraries, use parameter `status`. With this parameter, returned rows will
contain an additional field `liveStatus` that represents the status of the library.

- `0` means the library is closed.
- `1` means the library is open and has staff.
- `2` means the library is in self-service mode (no staff).

Parameter `status` can be used with OR without value.
- Value `open` returns libraries for which `liveStatus >= 1`.
- Value `closed` returns libraries for which `liveStatus = 0`.
- When value is omitted, result contains current day schedules for all libraries.

## Parameters
Name            | Description
--------------- | -----------
library         | List of library or service point IDs.
period.start    | Return results starting from this date. (`YYYY-MM-DD`)
period.end      | Return results until this date. (`YYYY-MM-DD`)
status          | Returns status status for libraries.

- When omitted, `period.start` and `period.end` default to `0d`.
- When using `status`, parameters `period.start` and `period.end` are ignored.

## Relative date ranges
Date ranges can be defined as exact dates or using relative values.

Unit of time  | Code
------------- | ----
Current days  | today
days          | d
weeks         | w
months        | m

When using these relative units, the reference point is always **current date**. Different units
behave in a different manner, also depending on whether they're used as the `start` or `end` value.

The intent is to produce "full calendar periods" i.e. using `week` as the unit results in ranges that
begin on Mondays and end on Sundays, whereas `month` results in ranges from the first to last day of
month. It is also possible to use one unit as the `start` value and another as `end` value.

- Negative values are also valid parameters.
- Weeks are considered to start on **Mondays** and end on **Sundays**.

## Some examples
Following examples use `2018-08-30` (Thursday) as the reference date.

Lower bound   | Upper bound   | Resulting period of time
------------- | ------------- | ------------------------
0d            | 0d            | 2018-08-30 – 2018-08-30
0w            | 0w            | 2018-08-27 – 2018-09-02
0m            | 0m            | 2018-08-01 – 2018-08-31
--            | --            | --
-1w           | 1w            | 2018-08-20 – 2018-09-02 (full weeks)
0m            | 2m            | 2018-08-01 – 2018-10-31 (full months)
0d            | 0w            | 2018-08-30 – 2018-09-02 (from today to end of week)

# Finna organisations
## Definition
```urls
https://api.kirjastot.fi/v4/finna_organisation
https://api.kirjastot.fi/v4/finna_organisation/<id>
```
This endpoint combines library consortiums and consortium-like "organisations" to be published
on Finna. In addition, each document comtains some extra fields.

Behaviour is similar to that of endpoint `/consortium`.

- If a consortium is not using Finna, it is not available via this endpoint.
- Filtering by Finna identifier is possible using `finna:id`.

## Additional data blocks
These blocks can be included into the results using parameter `with`.

Identifier      | Description
--------------- | -----------
links           | Links to websites related to this organisation.

Links contain a `category` field that can be used to group links into semantic groups.

# Finna service points
```urls
https://api.kirjastot.fi/v4/service_point
https://api.kirjastot.fi/v4/service_point/<id>
```

This endpoint combines libraries and non-library service points together. Behavior is similar to
endpoint `/library`, **however** non-library service points contain less fields, most notably
missing service and staff data.

- Here parameter `type` has additional possible values.

## Service point types
Type    | Description
------- | -----------
archive | Archive
museum  | Museum
other   | Unspecified type
