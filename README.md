# Puzzle & Dragons Monster Book

The monster book for [Puzzle & Dragons](http://www.gungho.jp/pad/).

You can see the generated article on [thisisgame.com](http://www.thisisgame.com/pad/info/monster/simulator.php?n=1743).

## Usage

For the task list:

``` sh
rake -T
```

Task        | Description
------------|---------------------------------------------------------------------
`update`    | Update the monster dictionary file by getting from puzzledragonx.com
`download`  | Download the monster icons from puzzledragonx.com
`grayscale` | Generate grayscale icons
`generate`  | Generate article based on article.yml
`diff`      | Print the difference of monsters in sort.yml and icon list
`rubocop`   | Run RuboCop

## License

Copyright (c) 2014 ChaYoung You. See [LICENSE.txt](LICENSE.txt) for details.

Copyright (c) 2014 [Puzzle Dragon X](http://www.puzzledragonx.com/) 4.00.

Puzzle & Dragons logo and all related images are registered trademarks or trademarks of [GungHo Online Entertainment, Inc](http://www.gunghoonline.com/games/puzzle-dragons/).
