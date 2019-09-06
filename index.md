# Exclave: Making Factory Tests Easy

Exclave is a factory test generation tool.  It is designed to reuse existing scripts that you have probably created during product development.  By allowing you to reuse existing programs, Exclave helps you to develop factory tests very quickly.

Exclave uses a Unix philosophy: Each test is a program that runs.  It prints its test progress to `stdout`, and any errors should go to `stderr`.  When a test is finished, it should either `exit 0` or, if a test fails, exit with a nonzero code.

## Simple Configuration

Every part of Exclave is configured from a Service file.  For example, here is a configuration for a test that simply runs `./validate-spi.sh` and fails if it either times out, or exits with a nonzero return code:

```ini
[Test]
ExecStart=./validate-spi.sh
Name=Validate SPI
Description=Validate that the SPI part number is what we expect
Timeout=1
```

Tests can depend on other tests.  Here is a test that loads the final firmware onto a device, but it depends on the tests passing:

```ini
[Test]
ExecStart=./fomu-flash -q -w pvt-top-multiboot.bin
Name=Load Final Bitstream
Description=Use fomu-flash to load the final bitstream
Requires=all-tests
Timeout=5
```

By building up test scenarios from these small building blocks, you can construct complicated tests to verify entire functionality of your product.

## Multiple Interfaces

What does a jig interface look like?  A test jig is a tool on a factory floor, and the interface depends on what you're testing.  If it's a laptop, you might have a TV attached with detailed reporting.  If it's a simple product, you might just have a PASS/FAIL indicator.

With Exclave, you can adapt the interface to meet the needs of the test jig.  An `interface` is a program that has bidirectional communication to Exclave.  They can receive line-oriented messages that reflect the state of a test.

An `interface` can be as simple as [a bash script that turns on LEDs](https://github.com/im-tomu/fomu-factory-test/blob/master/jig/bin/led-interface.sh#L94) or as complicated as a [full web server](https://github.com/exclave/jig-20-interface-http).

You can even specify multiple `interface` file in a single configuration.

## Raspberry Pi Image

We find that it's useful to use Raspberry Pis to drive the test jig.  They're cheap and easy to obtain when you want to build more testers.  And with 3.3V IO, they're able to drive JTAG and SWD directly.

A preconfigured Raspberry Pi image may be obtained from the [Exclave Pi Image](https://github.com/exclave/exclave-pi-gen/releases) repository.

In order to make it easy to remotely update the jig, and to improve jig lifetime, we mount the root filesystem as readonly.  Exclave and all of its support files are then run from a USB drive.  That way, all the factory workers need to do to update the jig is to unzip a file and replace the drive.

## Example Project

An example of using Exclave in production is the [Fomu](https://fomu.im/) [factory test](https://github.com/im-tomu/fomu-factory-test).  This repo is designed to be downloaded and loaded directly onto the root of a USB drive.  It is possible to send a link to [master.zip](https://github.com/im-tomu/fomu-factory-test/archive/master.zip) directly to the factory, which will result in a usable tester image.
