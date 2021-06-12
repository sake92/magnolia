![Magnolia](https://github.com/softwaremill/magnolia/raw/main/banner.jpg)

[<img alt="GitHub Workflow" src="https://img.shields.io/github/workflow/status/softwaremill/magnolia/Build/main?style=for-the-badge" height="24">](https://github.com/softwaremill/magnolia/actions)
[<img src="https://img.shields.io/badge/gitter-discuss-f00762?style=for-the-badge" height="24">](https://gitter.im/softwaremill/magnolia)
[<img src="https://img.shields.io/maven-central/v/com.softwaremill.magnolia/magnolia-core_3?color=2465cd&style=for-the-badge" height="24">](https://search.maven.org/artifact/com.softwaremill.magnolia/magnolia-core_3)

# Magnolia

__Magnolia__ is a generic macro for automatic materialization of typeclasses for datatypes composed from product types (e.g. case classes) and coproduct types (e.g. enums). It supports recursively-defined datatypes out-of-the-box, and incurs no significant time-penalty during compilation.

## Features

 - derives typeclasses for case classes, case objects and sealed traits
 - offers a lightweight syntax for writing derivations without needing to understand complex parts of Scala
 - builds upon Scala 3's built-in generic derivation
 - works with recursive and mutually-recursive definitions
 - supports parameterized ADTs (GADTs), including those in recursive types
 - supports typeclasses whose generic type parameter is used in either covariant and contravariant positions

## Getting Started

Given an ADT such as,
```scala
enum Tree[+T] derives Print:
  case Branch(left: Tree[T], right: Tree[T])
  case Leaf(value: T)
```
and provided an given instance of `Print[Int]` is in scope, and a Magnolia derivation for the `Print` typeclass
has been provided, we can automatically derive given typeclass instances of `Print[Tree[Int]]` on-demand, like
so,
```scala
Tree.Branch(Tree.Branch(Tree.Leaf(1), Tree.Leaf(2)), Tree.Leaf(3)).print
```
Typeclass authors may provide Magnolia derivations in the typeclass's companion object, but it is easy to create
your own.

The definition of a `Print` typeclass with generic derivation defined with Magnolia might look like this:
```scala
import magnolia.*

trait Print[T] {
  extension (x: T) def print: String
}

object Print extends AutoDerivation[Print]:
  def join[T](ctx: CaseClass[Typeclass, T]): Print[T] = value =>
    ctx.params.map { param =>
      param.typeclass.print(param.deref(value))
    }.mkString(s"${ctx.typeInfo.short}(", ",", ")")

  override def split[T](ctx: SealedTrait[Print, T]): Print[T] = value =>
    ctx.choose(value) { sub => sub.typeclass.print(sub.cast(value)) }
  
  given Print[Int] = _.toString
```

The `AutoDerivation` trait provides a given `autoDerived` method which will attempt to construct a corresponding typeclass
instance for the type passed to it. Importing `Print.autoDerived` as defined in the example above will make generic
derivation for `Print` typeclasses available in the scope of the import.

While any object may be used to define a derivation, if you control the typeclass you are deriving for, the
companion object of the typeclass is the obvious choice since it generic derivations for that typeclass will
be automatically available for consideration during contextual search.

If you don't want to make the automatic derivation available in the given scope, consider using the `Derivation` trait which provides semi-auto derivation with `derived` method, but also brings some additional limitations.
## Limitations

Magnolia is not currently able to access default values for case class parameters.

For a recursive structures it is required to assign the derived value to an implicit variable e.g.
```Scala
given instance: SemiPrint[Recursive] = SemiPrint.derived
```  
## Availability

For Scala 3:

```scala
val magnolia = "com.softwaremill.magnolia" %% "magnolia-core" % "2.0.0-M6"
```

For Scala 2, see the [legacy branch](https://github.com/softwaremill/magnolia/tree/legacy).

## Contributing

Contributors to Magnolia are welcome and encouraged. New contributors may like to look for issues marked
<a href="https://github.com/softwaremill/magnolia/labels/good%20first%20issue"><img alt="label: good first issue"
src="https://img.shields.io/badge/-good%20first%20issue-67b6d0.svg" valign="middle"></a>.

## Credits

Magnolia was originally designed and developed by [Jon Pretty](https://github.com/propensive), and is currently
maintained by [SoftwareMill](https://softwaremill.com).

## License

Magnolia is made available under the [Apache 2.0 License](/license.md).