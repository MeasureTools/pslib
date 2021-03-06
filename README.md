<!-- MarkdownTOC -->

- [Hitex PowerScale \(.psi/.psd\) Library](#hitex-powerscale-psipsd-library)
    - [Getting Started](#getting-started)
        - [Prerequisites](#prerequisites)
    - [How to load a .psi file](#how-to-load-a-psi-file)
    - [How to load the measurement data from a .psd file](#how-to-load-the-measurement-data-from-a-psd-file)
    - [How to write a .psi file](#how-to-write-a-psi-file)
    - [How to write .psd files](#how-to-write-psd-files)
    - [Running the tests](#running-the-tests)
    - [License](#license)
    - [Acknowledgments](#acknowledgments)

<!-- /MarkdownTOC -->

# Hitex PowerScale (.psi/.psd) Library

This is an open source library to read and write Hitex PowerScale (.psi/.psd) files. Aim of this project is to be 100% compatible with files generated by the Hitex PowerScale GUI Tool.

[![Build Status](https://travis-ci.org/MeasureTools/pslib.svg?branch=master)](https://travis-ci.org/MeasureTools/pslib)

## Getting Started

The simplest way to include this library to your project is by adding it as a gitsubmodule to your project.

```bash
git submodule add git@github.com:MeasureTools/pslib.git 3rdparty/pslib
```

Then add *pslib* or *pslib_s* to your CMake *target_link_libraries* and then include the wished pslib.h in your source code.

For example, to include the pslib for the psi/psd version 1.0 format use
```cpp
#include <pslib/pslib_v1_0.h>
```

### Prerequisites

This library requires [```boost```](http://www.boost.org/doc/libs/) and was tested with version **>=1.65**. It may work with earlier versions.

## How to load a .psi file

A *.psi* file can be loaded with the function ```pslib::v1_0::psi_t pslib::v1_0::load_psi(std::string)```.
This will only load the *.psi* file and **WONT** actually load the measurement data from the *.psd* files.
To load those see [How to load the measurement data from a .psd file](#how-to-load-the-measurement-data-from-a-psd-file)

```cpp
#include <pslib/pslib_v1_0.h>
#include <iostream>
#include <cstdlib>

int main(int argc, char* argv[]) {
    auto psi = pslib::v1_0::load_psi("example.psi");

    std::cout << "Filename:       " << psi.filename << std::endl;
    std::cout << "Checksum:       " << std::hex << psi.checksum << std::dec << std::endl;
    std::cout << "Sampling Rate:  " << psi.sampling_rate << std::endl;
    std::cout << "Sampling Count: " << psi.sampling_count << std::endl;
    std::cout << "----------- PROBES -----------" << std::endl;
    for (size_t i = 0; i < psi.probes.size(); ++i) {
        auto& probe = psi.probes[ i ];
        std::cout << "    " << "Probe[" << i << "].id:          " << probe.id << std::endl;
        std::cout << "    " << "Probe[" << i << "].port:        " << probe.port << std::endl;
        std::cout << "    " << "Probe[" << i << "].kind:        " << probe.kind << std::endl;
        std::cout << "    " << "Probe[" << i << "].voltage_min: " << probe.voltage_min << std::endl;
        std::cout << "    " << "Probe[" << i << "].voltage_max: " << probe.voltage_max << std::endl;
        std::cout << "    " << "Probe[" << i << "].current_min: " << probe.current_min << std::endl;
        std::cout << "    " << "Probe[" << i << "].current_max: " << probe.current_max << std::endl;
    }
    std::cout << "----------- PSDS -----------" << std::endl;
    for (size_t i = 0; i < psi.psds.size(); ++i) {
        auto& psd = psi.psds[ i ];
        std::cout << "    " << "PSDs[" << i << "].id:          " << psd.id << std::endl;
        std::cout << "    " << "PSDs[" << i << "].offset:      " << psd.offset << std::endl;
        std::cout << "    " << "PSDs[" << i << "].data_count:  " << psd.data_count << std::endl;
        std::cout << "    " << "PSDs[" << i << "].event_count: " << psd.event_count << std::endl;
    }

    return EXIT_SUCCESS;
}
```

## How to load the measurement data from a .psd file

```cpp
#include <pslib/pslib_v1_0.h>
#include <iostream>
#include <cstdlib>

int main(int argc, char* argv[]) {
    auto psi = pslib::v1_0::load_psi("example.psi");

    std::cout << "Samples:" << std::endl;
    auto samples = pslib::v1_0::load_samples(psi);
    for (size_t i = 0; i < samples.size(); ++i) {
        std::cout << "at(" << i << ")" << std::endl;
        auto sample = samples.at(i);
        std::cout << "    " << "time (ns): " << sample.time.count() << std::endl;
        for (auto& v : sample.values) {
            std::cout << "    " << "A:   " << v.current << std::endl;
            std::cout << "    " << "V:   " << v.voltage << std::endl;
        }
        for (auto& e : sample.events) {
            std::cout << "    " << "Event: " << e.data << std::endl;
        }
        std::cout << std::endl;
    }

    return EXIT_SUCCESS;
}
```

## How to write a .psi file

```cpp
#include <pslib/pslib_v1_0.h>
#include <limits>
#include <cstdlib>

int main(int argc, char* argv[])
{

    auto psi = pslib::v1_0::psi_t();
    {
        psi.filename = "example.psi";
        psi.checksum = 0;
        psi.sampling_rate = 1000;    // 1000 Hz
        psi.sampling_count = 1024;   // 1024 Samples

        // Add Probe
        auto probe = pslib::v1_0::probe_t();
        {
            probe.id = 1;   // id needs to be > 1 to be valid
            probe.port = 1; // port needs to be > 0 and < 4 to be valid
            probe.kind = pslib::v1_0::PROBE_KIND::STD;

            // If no min/max current or voltage is known set them to NaN
            probe.current_min = std::numeric_limits< double >::quiet_NaN();
            probe.current_max = std::numeric_limits< double >::quiet_NaN();
            probe.voltage_min = std::numeric_limits< double >::quiet_NaN();
            probe.voltage_max = std::numeric_limits< double >::quiet_NaN();
        }
        psi.probes.push_back(probe);

        // Add PSDFiles
        // A .psd file has a size of 1GiB and can hold a maximum number of 
        // samples per psd = size_per_sample / 1GiB with
        // size_per_sample = #probes * 2 * 8 Byte + #probes * 2 Byte + 2 Byte
        //
        // In this example:
        // #probes = 1
        // size_per_sample = 19 Byte
        // maximum number of samples per psd = ~17,695,128,917
        auto psd = pslib::v1_0::psd_t();
        {
            psd.id = 1;     // id needs to be > 1 to be valid
            psd.offset = 0; // Sum of data_count of each psd with a smaller id
            psd.data_count = 1024; // Number of Samples in this psd file
            psd.event_count = 0; // Number of events with event happend flag set
        }
        psi.psds.push_back(psd);
    }

    // write psi to file
    pslib::v1_0::save_psi(psi, "./", "example");

    // validate psi
    if (!pslib::v1_0::validate_psi(psi)) {
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```

## How to write .psd files

```cpp
#include <pslib/pslib_v1_0.h>
#include <cstdlib>
#include <chrono>

int main(int argc, char* argv[])
{

    auto psi = pslib::v1_0::psi_t();
    // [..] Same as in "How to write a .psi file"

    auto samples = pslib::v1_0::samples_t(psi, std::chrono::nanoseconds(0), psi.length());
    {
        for (size_t i = 0; i < psi.sampling_count; ++i) {
            for (size_t j = 0; j < psi.probes.size(); ++j) {
                auto ds = pslib::v1_0::data_stream_t();
                {
                    ds.current = double(i) / double(psi.sampling_count);
                    ds.voltage = double(psi.sampling_count) / double(i);
                }
                samples.values.push_back(ds);
            }
            for (size_t j = 0; j < (psi.probes.size() + 1); ++j) {
                auto e = pslib::v1_0::event_t();
                {
                    e.data = 0;
                }
                samples.events.push_back(e);
            }
        }
    }

    // write samples to psds
    pslib::v1_0::save_samples(samples, "./", "example");

    return EXIT_SUCCESS;
}
```

## Running the tests

To run the tests do the following:

```bash
mkdir build
cd build
cmake ..
make
make test
```

**Beware: The tests take upto 2GiB of RAM and 3GiB of Diskspace**

## License

This project is licensed under a modified BSD 1-Clause License with an additional non-military use clause - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

* TU Dortmund [Embedded System Software Group](https://ess.cs.tu-dortmund.de/EN/Home/index.html) which made this release possible
