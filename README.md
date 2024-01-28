# multitypes_third_and_fourth_approach_playground

/// =========================================================================================
/// Important - the main problem with the approach below is that f.e. you never cast Int to int you must use int property .value
/// to get acces to the real element. You would have to do entire intependent type system working seamlessly within itself
/// but have to sort of cast to int, String, etc. to be usable to real classes of dart
/// "Union" types - called by me multitypes, another approach THIS WORKS as expected, however:
/// ISSUE: you can only implement custom Int and built in native dart "Map" interface, real Map implementation class 
/// can be mixed with if it is top level class (or can't) but Int, Str can't be mixed so you have to 
/// SOLUTION: to avoid implementing Int each time you can use as of now (28.01.2024) experimantal feature
/// called macros:
/// best link: https://github.com/dart-lang/language/tree/main/working/macros/example
/// specs: https://github.com/dart-lang/language/blob/main/working/macros/feature-specification.md

sealed class Obj {}

/// Do understand if num is num or num is always double or int. For now let's do something like this; check int/double inheriting rules.
sealed class Num<T extends num> extends Obj {
  //late T _base;
  //get abc => _base;
}

sealed class Int extends Num<int> {
  int? _base;
  Int();
}

sealed class Dbl extends Num<double> {}

abstract class Str extends Obj {}

abstract class Map<K, V> extends Obj {}

// FIXME: WARNING PREVIOUS WRONG EXAMPLE
//sealed class IntOrStr implements Obj, Int {}
//
//class IntOrStrInt extends IntOrStr with IntMixin implements Int {}
//
//class IntOrStrStr extends IntOrStr with StrMixin implements Str {}

//
sealed class IntOrStr<K, V> implements Obj {}

class IntOrStrInt extends IntOrStr implements Int {
  // implementation good 
  int? _base;
  // also implementation good instead of property
  //set _base(int? value) => null;
  // also implementation good instead of property
  //get _base() => null;

}

class IntOrStrStr extends IntOrStr implements Str {}

///
sealed class IntOrMap<K, V> implements Obj {}

class IntOrMapInt extends IntOrMap implements Int {
  // implementation good int? _base;
  // also implementation good instead of property
  set _base(int? value) => null;
  // also implementation good instead of property
  get _base() => null;
}

class IntOrMapMap<K, V> extends IntOrMap<K, V> implements Map<K, V> {}

///

void abc(IntOrStr intOrStr) {
  // good as expected/wanted
  switch (intOrStr) {
    case Int():
      break;
    case Str():
      break;
  }
  // good as expected/wanted
  switch (intOrStr) {
    case IntOrStrInt():
      break;
    case IntOrStrStr():
      break;
  }
  // good as expected/wanted
  switch (intOrStr) {
    case Int():
      break;
    case IntOrStrStr():
      break;
  }

  // wrong as expected/wanted
  // switch (intOrStr) {
  //   case Int():
  //     break;
  //   case IntOrStrInt():
  //     break;
  // }

  //var abc = Int();
}

// for comparison this time the param is Obj
void abceeeee(Obj intOrStr) {
  // good, if missing Map() would be wrong as expected/wanted
  switch (intOrStr) {
    case Int():
      break;
    case Str():
      break;
    case Map():
  }
  //// good as expected/wanted
  //// Str is expected not IntOrStrStr()
  //switch (intOrStr) {
  //  case IntOrStrInt():
  //    break;
  //  case IntOrStrStr():
  //    break;
  //}
}

/// abceeeee accepts Obj not IntOrStr and IntOrMapInt() is Obj and Int in exhaustive checkig in abceeeee
void cdddddd() {
  abceeeee(IntOrMapInt());
}


sealed class SL {}

sealed class SL1 extends SL {
  SL1();
}

sealed class SL2 extends SL {
  SL2();
}

final class SL11 extends SL1 {
  SL11();
}

final class SL12 extends SL1 {
  SL12();
}

///////////////////////////////////////////////////////////
///////////////////////////////////////////////
////////////////////////////////////
///////////////////////////////
//////////////////////
///////////////

extension type EInt<T>(int i) implements int {

}

extension type EInt2<T>(EInt i) implements EInt {
}

// The point is that we can "hack" in an non-elegant much flawed way (int value is always there), to the point of doing 
// something closer to union types - if you do 10 - Eint3(...) it probably return Eint3 underlying int
// but with two types involved, f.e. intOrSomeClass where SomeClass is an extension type too imitating a class behaviour.
// "Never" causes the following code to stop executing, 
// @deprecated makes the code more visible and if Never is not possible everywhere
// @alwaysThrows - not needed
// Exhaustive switch() checking not possible
// But it seems that in a flawed way union types are "possible" for those basic types that cannot be extended nor implemented like int
// So possible only IntOrSomeClass (with not the second type like String, even Map that has implementable interface!) only not IntOr<T>
// However if you used code generation or macros you could
extension type EInt3<T>(EInt i) implements EInt {
  @deprecated
  Never operator <(int other) => throw Error;
}

extension type EInt4<T>(EInt i) implements EInt3 {
}


einttest(EInt eint) {
//  switch(eint) {
//    case Eint(): break,
//
//  }
}

einttest2(int eint) {
//  switch(eint) {
//    case Eint(): break,
//
//  }
}


einttestusagetest() {
  10 + EInt3(EInt2(EInt(10))); // unfortunately returns undrlying int value - cannot prevent it in any way
  // wrong as expected 
  // einttest(10);
  // good as expected and 10 + Eint(10) should work as far as i remember
  einttest(EInt(10));
  EInt(10) < 10;
  // ok:
  EInt2(EInt(20));
  // no:
  einttest(EInt2(EInt(20)));
  einttest(EInt4(EInt(20)));

  einttest2(10);
  einttest2(EInt(10));
  EInt(10) < 10;
  // ok:
  EInt2(EInt(20));
  // no:
  einttest2(EInt2(EInt(20)));
  einttest2(EInt4(EInt(20)));

  EInt2(EInt(20)) < EInt(10);
  EInt4(EInt(20)) < EInt(10); // "see Eint3 desc" "Never" causes the following code not to execute
  EInt3(EInt(20)) < EInt(10); // "see Eint3 desc" "Never" causes the following code not to execute
  // type Never of the < operator from EInt3 causes the below code to never be executed and is said to be "dead code".
  EInt3(EInt(20)) < EInt(10);
  EInt3(EInt(20)) < EInt(10);
}
