+++
type = "post"
date = 2018-09-10T15:51:12-05:00
draft = true
author = "Brainicorn"
title = "Ganno - A GO Annotation System"
description = "A library for building java-style annotation processors in golang at build time."
keywords = ["go", "golang", "library", "parser", "annotation", "annotations"]
topics = ["golang"]
tags = ["go", "annotation"]
github = "https://github.com/brainicorn/ganno"


+++

## Ganno - A GO Annotation System

Ganno is a GO library that parses java-style annotations from some input (usually source code comment blocks),
and returns structured data about the annotations that can be used to trigger some special functionality like code-generation.

This library was originally written to facilitate the creation of the [jsonschemagen](https://github.com/brainicorn/jsonschemagen) project
which parses go source code at build time and generates JSON-Schema files from GO structs.

Example annotations:

```go
@simpleAnnotation()

@simplewithParam(mykey=myval)

@quotedVal(somekey="my value")

@multipleParams(magic="wizards", awesome="unicorns")

@multipleVals(mypets=["dog", "kitty cat"])
```

Or you could split an annotation across multiple lines:

```go
// @stuffILike(
// 	instrument="drums"
// 	,mypets=[
// 		"dog"
// 		,"kitty cat"
// 	]
// 	,food="nutritional units"
// )
```

As a very basic example, let's feed the ganno parser a string and inspect it:

```go
input := `my @pet(name="fluffy buns", hasFur=true) is soooo cute!`

parser := ganno.NewAnnotationParser()

annos, errs := parser.Parse(input)

//there were some validation errors, we could inspect them, but let's just panic the first one
if len(errs) > 0 {
	panic(errs[0])
}

//since we know there's only one...
ourPet := annos.All()[0]

// get our pet's name
var name := "unknown"
if nameSlice, nameOk := ourPet.Attributes()["name"]; nameOk {
	name = nameSlice[0]
}

var hasFur := false
// get our pet's fur status
furSlice, furOk := ourPet.Attributes()["hasfur"]; if furOk {
	if hasFur, err := strconv.ParseBool(furSlice[0]); err != nil {
		hasFur = false
	}
}

fmt.Printf("%s is fluffy? %t\n", name, hasFur)
// Output: fluffy buns is fluffy? true
```

While the above example certainly works, there's a lot of validation that's going on that's not very reusable and is error prone. Let's see how we can use a custom factory to solve this...

By creating a custom annotation type and a factory for it, we can encapsulate all of the above validation logic and provide a cleaner interface when dealing with @pet annotations...

First off, let's create the "plugin" bits that could be put into their own package and/or library if desired....

```go
// PetAnno is a custom Annotation type for @pet() annotations
type PetAnno struct {
	Attrs map[string][]string
}

func (a *PetAnno) AnnotationName() string {
	return "pet"
}

func (a *PetAnno) Attributes() map[string][]string {
	return a.Attrs
}

// HasFur is a strongly-typed accessor for the "hasfur" attribute
func (a *PetAnno) Hasfur() bool {
	b, _ := strconv.ParseBool(a.Attrs["hasfur"][0])

	return b
}

// Name is an accessor for the "name" attribute
func (a *PetAnno) Name() string {
	return a.Attrs["name"][0]
}

// PetAnnoFactory is our custom factory that can validate @pet attributes and return new PetAnnos
type PetAnnoFactory struct{}

func (f *PetAnnoFactory) ValidateAndCreate(name string, attrs map[string][]string) (Annotation, error) {
	furs, furOk := attrs["hasfur"]
	_, nameOk := attrs["name"]

	// require the hasfur attr
	if !furOk {
		return nil, fmt.Errorf("pet annotation requires the attribute %q", "hasfur")
	}

	// require the name attr
	if !nameOk {
		return nil, fmt.Errorf("pet annotation requires the attribute %q", "name")
	}

	// make sure hasfur is a parsable boolean
	if _, err := strconv.ParseBool(furs[0]); err != nil {
		return nil, fmt.Errorf("pet annotation attribute %q must have a boolean value", "hasfur")
	}

	return &PetAnno{
		Attrs: attrs,
	}, nil
}
```

With all of that in place, our dealings with @pet annotations is simplfied:

```go
parser := NewAnnotationParser()

//register our factory
parser.RegisterFactory("pet", &PetAnnoFactory{})

input := `my @pet(name="fluffy buns", hasFur=true) is soooo cute!`

annos, errs := parser.Parse(input)

if len(errs) > 0 {
	panic(errs[0])
}

// get our one pet annotation and cast it to a PetAnno
mypet := annos.ByName("pet")[0].(*PetAnno)

fmt.Printf("%s is fluffy? %t\n", mypet.Name(), mypet.Hasfur())
// Output: fluffy buns is fluffy? true
```

For more information, check out [The Ganno github repository](https://github.com/brainicorn/ganno)
