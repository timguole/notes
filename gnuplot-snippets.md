# gnuplot snippets

## Call script with arguments

Pass arguments to gnuplot script

```shell
gnuplot -c SCRIPT ARG1 ARG2 ... ARGn
```

Use arguments in gnuplot scripts

```shell
set output AG1
set title ARG2
```

## Snippets

#### Plot time serial data from a file

data.file

```tex
202104201001 3
202104201002 5
202104201003 2
```

plot commands

```shell
set terminal png size 800,400
set output 'foo.png'
set title 'Foo Bar'
set xdata time
set format x '%H:%M'
set timefmt '%Y%m%d%H%M'
set yrange [0:1000]
plot 'file.data' using 1:2 with boxes
```

#### Plot with two Y-axes

> This snippet assumes that the first line in file.data is column names.

data.file

```tex
Time       C1 C2 C3
202104201001 3 2 4
202104201002 5 6 8
202104201003 2 4 5
```

plot command

```shell
set terminal png size 1800,1000
set output 'foo.png'
set key autotitle columnhead

set xdata time
set format x '%d%H%M'
set timefmt '%y%m%d%H%M%S'
set xlabel 'Time'

set yrange [0:100]
set ylabel 'Y Label'

set ytics nomirror
set y2tics 100,1
set y2range [100:128]
set y2label 'Y2 Label'

plot 'file.data' using 1:2 with line axis x1y1, file.data using 1:3 with line axis x1y2
```

