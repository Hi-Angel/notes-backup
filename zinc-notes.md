## Misc

Decision vars are constraints, and are written like

    var int: i; constraint i >= 0; constraint i <= 4;

It's equivalent to:

    var 0..4: i;

The following 2 lines are too equivalent:

    var int: i = x + 3;
    var int: i; constraint i = x + 3;

Some examples with enums:

    enum color = {green, blue, pink, yellow};
    var color: A;
    var color: B;
    var color: C;
    constraint A != B;
    constraint C != A;

## Array:

    array[someSize1] of int: myArr;
    array[someSize2] of var int: myObjectiveArr;
    profit = [400, 500]; % 1d array initialization

Multidimensional array initialization:

    set of int: COL = 1..5;
    set of int: ROW = 1..2;
    array[ROW,COL] of int: c =
                    [| 250, 2,  75, 100,  0
                     | 200, 0, 150, 150, 75 |];

arrays always have either implicit or explicit index set. Explicit is obvious: e.g. you can make an array to be enumerated with a `enum`. The implicit is the caveat: default index set starts with `1`. So e.g. you can't do:

       array[0..4] of int: readings = [35,35,20,20,20];

because index set of `readings` explicitly starts with `0` but the one of `[35,..20]` implicitly starts with `1`. A workaround:

       array[0..4] of int: readings = array1d(0..4, [35,35,20,20,20]);

Note: you can also use `array[int] of int: readings = [35,65]`, this way index array is inherited.

## Set

Sets are commonly used as index sets *(see math index set)*

    var set of 1..NrI : Items;

Set builder:

    [space[i,j] | i in Rows, j in Columns]

## Constraints:

    constraint forall(i in myEnum)(myArr[i] <= 0);
    constraint sum(i in myEnum)(size[i] * myArr[i]) <= capacity;
    constraint alldifferent ([M1,M2,M3,M4,M5,M6]);

## Solve:

    solve maximize sum(i in myEnum)(a[i] * b[i]);

or

    solve satisfy;

## strings:

`++` is string concatenation

`show(v)` built-in func to show the value of `v`

`"\(v)"` means substitue the value of `v` into the string.

## output example:

    output [show(sum(myarr)), " = \(myarr)"];

## caveats
