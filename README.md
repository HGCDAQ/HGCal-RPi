# HGCal-RPi:link-test

This branch contains code and firmware intended for testing connections between readout and sync boards.
It should be considered entirely separate from the other branches.

# Instructions
There are two main tests that can be run.

## RDOUT\_DONE Counting
This test is intended to test whether or not `RDOUT_DONE` signals sent through IPBus are correctly propagated through the readout board.
It can also show if the `RDOUT_DONE` signals sent from the readout board are not recieved by the sync board.

An executable is started up on the syncboard which shows the number of times it has recieved a `RDOUT_DONE` signal from each readout board.
IPBus then sends the `RDOUT_DONE` signal on a readout board (using the `test_done_count.py` script).
There is a special `RDOUT_DONE_COUNT` register on the readout board to ensure the signal was propagated through the board correctly.
This register is checked before sending `RDOUT_DONE` and after, and if it doesn't increment by one, the script exits.
This process is repeated many times.

If the python script exits before sending all 65535 signals, this means one of the `RDOUT_DONE` signals got lost inside the readout board after being sent by IPBus.
If instead the python script finishes but the `rdout_done_count` on the sync board does not match the number of signals sent, it is likely that a signal was lost between the readout board and the sync board.
If this happens, you can use prbs checking to test the rdout-sync link further.
If the python script finishes and the `rdout_done_count` on the sync board does match the number of signals sent, the link should be OK.

To run this test, first edit `copy-dirs` with the correct ssh aliases for the readout and sync boards you will be using.
I will reference these aliases in the rest of the instructions as `SYNC_ALIAS` and `RDOUT_ALIAS`.
  1. Run the `copy-dirs` script to copy the software over to the readout and sync boards.
  2. ssh into the readout board, and change to the copied directory: `cd ~/linktest-rdout/`
  3. Open a new terminal window, ssh into the sync board, and change into the copied directory: `cd ~/linktest-sync/`
  4. Run the setup scripts with `./run_rdout` and `./run_sync`, respectively. This compiles the software and programs the ORMs.
      - The `sync_debug` executable will then start up on the sync board. You should see error and readout done counts for each cable on the sync board: `loop =   0, rdout_done_count[ 0] =     0` and `loop =   0, prbs_error_count[ 0] =    0`.
  5. Exit from the readout board. Setup IPBus using `source etc/env.sh`.
  6. Run `test_done_count.py`. On the sync board screen, you should see the trigger count for the connected cable increase.
  7. Once the script finishes, change cable location on the sync board and repeat the test.

## PRBS Checking
