Collective communication 2
==========================

.. questions::

   - How can I split data across the ranks of my program?
   - How can I join data from the ranks of my program?

.. objectives::

   - Know the difference between scatter and gather


Scatter
-------

An ``MPI_Scatter`` call sends data from one rank to all other ranks.


.. figure:: img/MPI_Scatter.svg
   :align: center

   After the call, all ranks in the communicator have the one value
   sent from the root rank, ordered by rank number.

``MPI_Scatter`` is `blocking` and introduces `collective
synchronization` into the program.

This can be useful to allow one rank to share values to all other
ranks in the communicator. For example, one rank might compute some
values, and then scatter the content to all other ranks. They can then
use this as input for future work.

Call signature::

  int MPI_Scatter(const void *sendbuf, int sendcount, MPI_Datatype sendtype,
                  void *recvbuf, int recvcount, MPI_Datatype recvtype,
                  int root, MPI_Comm comm)

Link to `MPI_Scatter man page <https://www.open-mpi.org/doc/v4.0/man3/MPI_Scatter.3.php>`_

Link to `Specification of MPI_Scatter <https://www.mpi-forum.org/docs/mpi-3.1/mpi31-report/node105.htm#Node105>`_

.. note::

   All ranks must supply the same value for ``root``, which specifies
   the rank of that communicator that provides the values that are
   sent to all other ranks.

   ``sendbuf``, ``sendcount`` and ``sendtype`` describe the buffer on
   the **root** process from which the data comes. ``sendcount``
   describes the extent of the buffer sent to each other rank, not the
   extent of the whole buffer! Other ranks do not need to allocate a
   send buffer, and may pass any values to the call.

   ``recvbuf``, ``recvcount`` and ``recvtype`` describe the buffer on
   **each** process to which the data is sent. Only a buffer large
   enough to receive the data for that process is needed.

   All ranks in the communicator must participate with valid receive
   buffers and consistent counts and types.

Gather
------

An ``MPI_Gather`` call sends data from all ranks to a single rank.
It is the inverse operation of ``MPI_Scatter``.

.. figure:: img/MPI_Gather.svg
   :align: center

   After the call, the root rank has one value from each other rank in
   the communicator, ordered by rank number.

``MPI_Gather`` is `blocking` and introduces `collective
synchronization` into the program.

This can be useful to allow one rank to collect values from all other
ranks in the communicator. For example, all ranks might compute some
values, and then the root rank gathers the content. It can then use
this as input for future work. One use case is to combine data so that
one rank can compute a combined property, or write all the data to a
file.

Call signature::

  int MPI_Gather(const void *sendbuf, int sendcount, MPI_Datatype sendtype,
                 void *recvbuf, int recvcount, MPI_Datatype recvtype,
                 int root, MPI_Comm comm)

Link to `MPI_Gather man page <https://www.open-mpi.org/doc/v4.0/man3/MPI_Gather.3.php>`_

Link to `Specification of MPI_Gather <https://www.mpi-forum.org/docs/mpi-3.1/mpi31-report/node103.htm#Node103>`_

.. note::

   All ranks must supply the same value for ``root``, which specifies
   the rank of the process within that communicator that receives the
   values send from each process.

   ``sendbuf``, ``sendcount`` and ``sendtype`` describe the buffer on
   **each** process from which the data is sent. Only a buffer large
   enough to contain the data sent by that process is needed.

   ``recvbuf``, ``recvcount`` and ``recvtype`` describe the buffer on
   the **root** process in which the data is received. ``revcount``
   describes the extent of the buffer received from each rank, not the
   extent of the whole buffer! Other ranks do not need to allocate a
   receive buffer, and may pass any values to the call.

   All ranks in the communicator must participate with valid send
   buffers and consistent counts and types.

 
Code-along exercise: scatter and gather
---------------------------------------

.. challenge:: 2.1 Use a scatter and gather

   1. Download the :download:`source code
      <code/collective-communication-scatter-and-gather.c>`. Open
      ``collective-communication-scatter-and-gather.c`` and read
      through it. It's similar to the broadcast code we saw
      earlier. Try to compile with::

        mpicc -g -Wall -std=c11 collective-communication-scatter-and-gather.c -o collective-communication-scatter-and-gather

   2. When you have the code compiling, try to run with::

        mpiexec -np 4 ./collective-communication-scatter-and-gather

   3. Use clues from the compiler and the comments in the code to
      change the code so it compiles and runs. Try to get all ranks to
      report success :-)

.. solution::

   * One correct pair of calls is::

         MPI_Scatter(values_to_scatter, 1, MPI_FLOAT,
                     &scattered_value, 1, MPI_FLOAT,
                     rank_of_scatter_root, comm);
         /* ... */
         MPI_Gather(&result, 1, MPI_FLOAT,
                    gathered_values, 1, MPI_FLOAT,
                    rank_of_gather_root, comm);

   * What happened if you mistakenly used 4 for the scatter send count or
     the gather receive count. Why?
   * Download a :download:`working solution <code/collective-communication-scatter-and-gather-solution.c>`


All-gather
----------

An ``MPI_Allgather`` call gather the same data from all ranks and
provides it to all ranks. It is logically identical to ``MPI_Gather``
to a root followed by an ``MPI_Bcast`` from that root, but is
implemented more efficiently.

.. figure:: img/MPI_Allgather.svg
   :align: center

   After the call, all ranks have one value from each other rank in
   the communicator, ordered by rank number.

``MPI_Allgather`` is `blocking` and introduces `collective
synchronization` into the program. Note that there is no root
for this operation.

This can be useful to allow all ranks to collect values from all other
ranks in the communicator. For example, all ranks might compute some
values, and then all ranks gather that content to use it in a
subsequent stage.

Call signature::

  int MPI_Allgather(const void *sendbuf, int sendcount, MPI_Datatype sendtype,
                    void *recvbuf, int recvcount, MPI_Datatype recvtype,
                    MPI_Comm comm)

Link to `MPI_Allgather man page <https://www.open-mpi.org/doc/v4.0/man3/MPI_Allgather.3.php>`_

Link to `Specification of MPI_Allgather <https://www.mpi-forum.org/docs/mpi-3.1/mpi31-report/node107.htm#Node107>`_

.. note::

   All ranks receive the values send from each process.

   ``sendbuf``, ``sendcount`` and ``sendtype`` describe the buffer on
   **each** process from which the data is sent. Only a buffer large
   enough to contain the data sent by that process is needed.

   ``recvbuf``, ``recvcount`` and ``recvtype`` describe the buffer on
   **each** process to which the data is sent. A buffer large
   enough to receive all the data for that process is needed.

   All ranks in the communicator must participate with valid receive
   buffers and consistent counts and types.


All-to-all
----------

An ``MPI_Alltoall`` call gathers data from all ranks and provides
distinct data to all ranks. It is logically identical to making one
call to ``MPI_Gather`` for each possible root rank, but is implemented
more efficiently.

.. figure:: img/MPI_Alltoall.svg
   :align: center

   After the call, all ranks have one value from each other rank in
   the communicator, ordered by rank number.

``MPI_Alltoall`` is `blocking` and introduces `collective
synchronization` into the program. Note that there is no root
for this operation.

This can be useful to allow all ranks to collect values from all other
ranks in the communicator. For example, a 3D Fast Fourier Transform
often uses an all-to-all operation to redistribute the working data
set for each process to a new dimension.

Call signature::

  int MPI_Alltoall(const void *sendbuf, int sendcount, MPI_Datatype sendtype,
                   void *recvbuf, int recvcount, MPI_Datatype recvtype,
                   MPI_Comm comm)

Link to `MPI_Alltoall man page <https://www.open-mpi.org/doc/v4.0/man3/MPI_Alltoall.3.php>`_

Link to `Specification of MPI_Alltoall <https://www.mpi-forum.org/docs/mpi-3.1/mpi31-report/node109.htm#Node109>`_

.. note::

   All ranks receive a subset of the values sent from each process.

   ``sendbuf``, ``sendcount`` and ``sendtype`` describe the buffer on
   **each** process from which the data is sent. Only a buffer large
   enough to contain the data sent by that process is needed.

   ``recvbuf``, ``recvcount`` and ``recvtype`` describe the buffer on
   **each** process to which the data is sent. A buffer large
   enough to receive all the data for that process is needed.

   All ranks in the communicator must participate with valid receive
   buffers and consistent counts and types.


Code-along exercise: all-gather and all-to-all
----------------------------------------------

.. challenge:: 3.1 Use all-gather

   1. Download the :download:`source code
      <code/collective-communication-allgather.c>`. Open
      ``collective-communication-allgather.c`` and read
      through it. It's similar to the broadcast code we saw
      earlier. Try to compile with::

        mpicc -g -Wall -std=c11 collective-communication-allgather.c -o collective-communication-allgather

   2. When you have the code compiling, try to run with::

        mpiexec -np 4 ./collective-communication-allgather

   3. Use clues from the compiler and the comments in the code to
      change the code so it compiles and runs. Try to get all ranks to
      report success :-)

.. solution::

   * One correct call is::

         MPI_Allgather(values_to_all_gather, 3, MPI_INT,
                       &all_gathered_values, 3, MPI_INT,
                       comm);

   * What happened if you mistakenly used 4 or 12 for the counts? Why?
   * Download a :download:`working solution <code/collective-communication-allgather-solution.c>`

.. challenge:: 3.2 Use all-to-all

   1. Download the :download:`source code
      <code/collective-communication-alltoall.c>`. Open
      ``collective-communication-alltoall.c`` and read
      through it. It's similar to the broadcast code we saw
      earlier. Try to compile with::

        mpicc -g -Wall -std=c11 collective-communication-alltoall.c -o collective-communication-alltoall

   2. When you have the code compiling, try to run with::

        mpiexec -np 4 ./collective-communication-alltoall

   3. Use clues from the compiler and the comments in the code to
      change the code so it compiles and runs. Try to get all ranks to
      report success :-)

.. solution::

   * One correct call is::

        MPI_Alltoall(values_to_all_to_all, 3, MPI_INT,
                     &result_values, 3, MPI_INT,
                     comm);

   * What happened if you mistakenly used 4 or 12 for the counts? Why?
   * Download a :download:`working solution <code/collective-communication-alltoall-solution.c>`

See also
--------

* Upstream information
* Another course



.. keypoints::

   - TODO
   - point 2
   - ...
