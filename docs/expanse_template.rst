..
  SPDX-FileCopyrightText: 2026 SeisSol Group

  SPDX-License-Identifier: BSD-3-Clause
  SPDX-LicenseComments: Full text under /LICENSE and /LICENSES/

  SPDX-FileContributor: Author lists in /AUTHORS and /CITATION.cff Isabel Reinicke

.. _compile_run_expanse:


Expanse-SDSC
============

To compile SeisSol using Spack modules on Expanse-SDSC (San Diego Supercomputer Center), follow the procedure below.


Install Spack
-------------

Follow the steps for the spack installation here `Installation with Spack <https://github.com/SeisSol/seissol-spack-aid/blob/main/spack/README.rst>`_
which should include:

Load necessay modules
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    module load gcc/10.2.0
    module load openmpi/mlnx/gcc/64/4.1.5a1
    module load cmake/3.21.4

Clone the `Spack repo <https://github.com/spack/spack.git>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

  git clone --depth 1 --branch v0.21.1 https://github.com/spack/spack.git
  cd spack


Modify  ~/.spack/packages.yaml 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

  packages:
   openmpi:
    externals:
    - spec: openmpi@4.1.5
     prefix: /usr/mpi/gcc/openmpi-4.1.5a1
    buildable: False

Clone the `seissol-spack-aid repo <https://github.com/SeisSol/seissol-spack-aid.git>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

  cd
  git clone --recursive https://github.com/SeisSol/seissol-spack-aid.git
  cd $HOME/seissol-spack-aid
  
Make SeisSol visible for Spack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

  cd $HOME/seissol-spack-aid
  spack repo add ./spack
  


Type the following to see all compilers avaliable for Spack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: bash

  cd
  spack compiler list


Make sure that you have C/C++ and Fortran compilers in your compiler collection
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

  cat $HOME/.spack/linux/compilers.yaml 
This should show

.. code-block:: bash

  ...
  paths:
    cc: /usr/bin/gcc
    cxx: /usr/bin/g++
    f77: /usr/bin/gfortran
    fc: /usr/bin/gfortran
  ...

Install the packages you will need for the SeisSol installation (using spack)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: bash

  spack install --add cmake
  spack install python
  spack install py-numpy
  spack install py-scipy

Update your ``~/.bashrc`` file as follows (or create a ``setup_spack_seissol.sh`` file including)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: bash

  source /home/<user>/spack/share/spack/setup-env.sh
  module load gcc/10.2.0
  module load openmpi/mlnx/gcc/64/4.1.5a1
  module load cmake/3.21.4
    
After sourcing your ``~/.bashrc`` (or ``setup_spack_seissol.sh``), you should be able to use spack for all steps of the SeisSol installation.

|
|

Install SeisSol
---------------
.. code-block:: bash

  spack install seissol-env +mpi +asagi %gcc@10.2.0 ^openmpi@4.1.5

Load necessary modules with spack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: bash

  spack load cmake
  spack load python
  spack load py-numpy
  spack load py-scipy

Clone `SeisSol <https://github.com/SeisSol/SeisSol.git>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: bash

  git clone --recursive https://github.com/SeisSol/SeisSol.git
  cd SeisSol
  mkdir build-release && cd build-release



Configure compilation variables 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
(here example with Release configuration, double precision and polynomial order of degree 4)

.. code-block:: bash

  CC=mpicc CXX=mpiCC FC=mpif90 cmake -DNUMA_AWARE_PINNING=ON -DASAGI=ON -DCMAKE_BUILD_TYPE=Release -DHOST_ARCH=rome -DPRECISION=double -DORDER=4 -DGEMM_TOOLS_LIST=PSpaMM ..


Compile SeisSol
^^^^^^^^^^^^^^^
.. code-block:: bash

  make -j64

Update your ``~/.bashrc`` (or your ``setup_spack_seissol.sh``) file as follows
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

  # Needed for spack 
  source /home/<user>/spack/share/spack/setup-env.sh
  
  module load gcc/10.2.0
  module load openmpi/mlnx/gcc/64/4.1.5a1
  module load cmake/3.21.4
  
  
  # Needed for SeisSol
  export SPACK_ROOT=/home/<user>/spack
  export PATH=$SPACK_ROOT/bin:$PATH
  
  spack load seissol-env
  spack load cmake
  
  spack load python
  spack load py-numpy
  spack load py-scipy

After sourcing your ``~/.bashrc`` (or ``setup_spack_seissol.sh``), you should be able to recompile SeisSol if needed (e.g. if you want to change DPRECISION or DORDER).

To run SeisSol you can either link the executable (from your scratch file/where your job.sh is)

.. code-block:: bash

  ln -s /home/<user>/SeisSol/build-release/SeisSol_Release_drome_4_viscoelastic2


or set its path explicitly **in the slurm file** (job.sh)

.. code-block:: bash

  # prepare environment to run Seissol (Spack)
  source ~/setup_spack_seissol.sh

  # set executable explicitly
  export SEISSOL_EXE=$HOME/SeisSol/build-release/SeisSol_Release_drome_4_viscoelastic2

You can then run SeisSol (e.g. using this in your slurm file job.sh)

.. code-block:: bash

  srun --mpi=pmix $SEISSOL_EXE parfile.par
