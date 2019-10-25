### Setup AWS

- `scp chazz`
- `scp ~/.aws`
- `scp ~/.ssh/*.pem`

### How to Run Spatial on AWS

- Spatial requires [`pkg-config, libisl, libbmp`](https://github.com/stanford-ppl/spatial/issues/186). 
  - CentOS: 
    - CentOS cannot install these package with `gcc`. Compiling from source and `download_prerequisites`also get stuck for I don't know why. Let's just download and compile.
    - `yum install gmp-devel`; (this must be first step, otherwise libisl cannot find `bmp.h`)
    - download [libisl](http://isl.gforge.inria.fr/), install with `./configure && make && sudo make install`. Default installed in `/usr/local/lib/`. So add `export LD_LIBRARY_PATH=/usr/local/lib ` to `~/.bash_profile`.
    - `pkg-config`: (`sudo yum -y install glib-devel`.)  download [pkg-config](http://pkgconfig.freedesktop.org/releases/pkg-config-0.28.tar.gz) and `./configure --with-internal-glib && make && sudo make install`.
  - I love Ubuntu
  
- Simulation should work now.

  - `sbt`(need to install `java` and then `sbt`)
  - `compile`
  - `runMain HelloSpatial --sim`

- Set up aws-fpga (use version 14, the server seems have strange version)

  - ```shell
    export AWS_HOME=~/spatials/aws-fpga/
    source $AWS_HOME/hdk_setup.sh
    sudo source $AWS_HOME/sdk_setup.sh
    ```

- Compile 

  - `export SPATIAL_HOME=`pwd`/spatial-quickstart`
  - `export DATA_HOME=`pwd`/spatial-lang/apps/data/`; modify `$DATA` to `sys.env("DATA_HOME")` 
  - `sbt`
  - `runMain appName --synth --fpga=AWS_F1` 

- Generate [bitstream](https://github.com/stanford-ppl/spatial/blob/09c028b754e935eefcd55c599cedd7ebbd71fa32/resources/synth/aws.Makefile)

  - goto `gen/appName` and change `build.sbt`: scalatest 2.2.5 => 3.05; scalacheck 1.12.4 => 1.13.4

  - `java.lang.OutOfMemoryError` => `export _JAVA_OPTIONS="-Xmx4g"` (increase heap)

  - `make aws-F1-afi | tee make.log` 

    

