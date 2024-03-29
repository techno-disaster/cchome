---
title: "Python extension module for CCExtractor"
---

This is the main documentation of Python extension module for
CCExtractor:

##### CCExtractor Library

#### Refactoring the codebase into a library

Earlier version of CCExtractor was compiled as a binary and could not be
used as a library. The entire codebase was executed via a single main
function defined in ccextractor.c and this architecture was not suitable
for extending ccextractor source code to a library. Hence, many
modifications were made to ccextractor.c so that conversion to a library
could be done. Major modifications were:

- Segmenting the larger functions into smaller functions so that they could be called from one main function. Earlier the entire processing was carried out from one main function itself. This was not a good idea considering the possibility for library. This would allow the user to set the parameters to be passed to CCExtractor from Python with one parameter at a time and not the entire list of all parameters together.
- The refactoring of the code base and architectural judgements as to how the code should be segmented so that the entire working remains the same and also the library structure could be established.

Apart from these changes, the header file ccextractor.h was also
included into the codebase to define many global variables as well as
the function declarations of definitions made in ccextractor.c. The
major changes could be seen at this [PR (merged)](https://github.com/CCExtractor/ccextractor/pull/744).
However, following the next stages of development after the changes made
in the above mentioned PR, the final structure could be found at
[ccextractor.c](https://github.com/CCExtractor/ccextractor/blob/master/src/ccextractor.c)
and
[ccextractor.h](https://github.com/CCExtractor/ccextractor/blob/master/src/ccextractor.h).

#### Definitions made in ccextractor.h

 * In ccextractor.h, the major changes included declaring global variables which would be accessible throughout the codebase for calling the respective callbacks (discussed later in the documentation) from C to Python for processing the caption frames in Python as they are extracted in CCExtractor. The global variable was also used to keep a track of the start time and end time of caption frames so that the CE-608 grids belonging to the same frame could be clubbed together. The global variable [array](https://github.com/CCExtractor/ccextractor/blob/master/src/ccextractor.h#L46) has been defined and the [respective structure](https://github.com/CCExtractor/ccextractor/blob/master/src/ccextractor.h#L35) definition has also been done in ccextractor.h.
 * The global variable array is an instance of this [structure](https://github.com/CCExtractor/ccextractor/blob/master/src/ccextractor.h#L35). The elements of the structure are PyObject* reporter, int sub_count and struct python_subs_modified* subs. The [subs](https://github.com/CCExtractor/ccextractor/blob/master/src/ccextractor.h#L30) has been defined with start_time and end_time as its elements. More detailed description about why start_time and end_time have been used is given in the section describing about extractors. The main motivation for defining a global variable to catch hold of start time and end time of the caption frames as they are processed in CCExtractor is to identify the text, font and color grids (for CE-608 captions) that belong to the same caption frame.
 * The major point to note is that the compilation of Python extension module includes [setting a macro](https://github.com/CCExtractor/ccextractor/blob/master/api/build_api#L19) PYTHONAPI  which acts as an indication that the compilation is made for Python extension module and this helps in declaring as well as defining the functions which are only needed for Python extension module. As defined [here](https://github.com/CCExtractor/ccextractor/blob/master/src/ccextractor.h#L36), the PYTHONAPI macro is used to define the functions/variables which are needed only by the extension module. 
 * Another major advantage of defining the macro PYTHONAPI is that the definitions made for Python extension module only need python-dev package as a prerequisite for compilation. However, if the user wants to compile only CCExtractor and not the Python extension module, then the code should not have python-dev package as a dependency. This has been attained by using macro PYTHONAPI and C pre-processors.

##### CCExtractor Python Extension Module

#### Extension module dependencies

### 1. SWIG

 * For generation of the wrappers of the C code base, which would then be used to compile the extension module, I have used [SWIG](http://www.swig.org/) (swig-3.0.12). The entire compilation has been included in a build script (discussed later) and the user need not have prior knowledge of SWIG to get started.
 * For compiling the Python extension module, the second dependency in addition to the dependencies of CCExtractor is SWIG. The user can follow these [installation steps](http://www.swig.org/download.html) for getting SWIG installed.
 * For generating the wrappers of the C/C++ code in a user required language, the user needs to have a basic understanding of the [interface file](http://www.swig.org/Doc3.0/Introduction.html#Introduction_nn5) which is used by SWIG. However, in case of generating the extension module for CCExtractor, the interface file has been written and is available [here](https://github.com/CCExtractor/ccextractor/blob/master/api/ccextractor.i). SWIG uses this interface file to generate the wrappers for CCExtractor which are then compiled to form the extension module.

### 2. Python-dev package

#### Overall architecture

 * The entire Python Extension module related work is done in the [api/](https://github.com/CCExtractor/ccextractor/tree/master/api) directory with modifications to the CCExtractor codebase to integrate the divergent path, CCExtractor would take if the processing is done via Python module.

#### Generating the Python extension module

 * For this project, I have mainly used two build scripts, viz., [build_api](https://github.com/CCExtractor/ccextractor/blob/master/api/build_api) and [build_library](https://github.com/CCExtractor/ccextractor/blob/master/api/build_library) which are both present in the api/ directory. For generating the Python bindings, user need to just run the build_library script as ./build_library. This would internally generate the SWIG wrappers from the SWIG interface file (ccextractor.i) present in the same directory. The user should note that if the user has not installed SWIG, the the compilation would stop at this step itself. Once the wrappers are generated, then the build_library script  would execute the build_api script which would compile the entire code base of CCExtractor along with the wrappers generated by SWIG. In addition to this, build_api would also compile the extractors and wrappers defined in the extractors/ and wrappers/ directories respectively. Once the compilation is successful, then build_library would generate a shared library called _ccextractor.so from the entire code which would be shared object for the module.
 * In addition to generating the  wrapper codes generated by SWIG, it also outputs the ccextractor.py which would be later used as Python extension module for accessing CCExtractor functionality via Python.
 * As mentioned in earlier section, the build_api compiles the entire code base with an option -DPYTHONAPI which is used by GCC to define a macro PYTHONAPI. This macro then acts as a signal telling that the extension module is being generated and the bindings dependency need a check as well as the bindings dependent functions need to be defined.

#### Workflow of Python extension module

The following section encompasses on the detailed description of the
entire workflow of Python extension modules and the importance of each
function in the codeflow. An example usage has been done in
[api_testing.py](https://github.com/CCExtractor/ccextractor/blob/master/api/api_testing.py).

### api_init_options

Function declaration- **struct ccx_s_options* api_init_options()**

 * This function returns an initialized instance of struct ccx_s_options which is modified in CCExtractor according to the values of the parameters provided by the user while executing CCExtractor.

### check_configuration_file

Function declaration- **void check_configuration_file(struct ccx_s_options api_options)** This function is used to check the
configuration file and it takes the struct ccx_s_options instance as
returned by api_init_options().

### api_add_param

Function declaration- **void api_add_param(struct ccx_s_options* api_options,char* arg)**

 * The api_add_param function is used to add user passed parameters to the struct ccx_s_options instance which would be used to compile the parameters and make the necessary modifications in the working of CCExtractor.
 * This function takes the instance of struct ccx_s_options passed to check_configuration_file function and also, the string denoting the parameter passed by the user.
 * The parameters are added to the [python_params](https://github.com/CCExtractor/ccextractor/blob/master/src/lib_ccx/ccx_common_option.h#L198) element of struct ccx_s_options and the count of the parameters is kept in [python_param_count](https://github.com/CCExtractor/ccextractor/blob/master/src/lib_ccx/ccx_common_option.h#L199).

### my_pythonapi

Function declaration- **(depends on whether the compilation is done as CCExtractor binary or as extension modules)**

 * The my_pythonapi is defined on the basis of how the compilation has been done by the user. If the user wants to use CCExtractor binary rather than the Python extension module, then this function is defined as #define my_pythonapi(args, func) set_pythonapi(args)  with the set_pythonapi being defined in [wrapper.c](https://github.com/CCExtractor/ccextractor/blob/master/api/wrappers/wrapper.c#L4).
 * However, if the user wants to build the extension module, then the definition of my_pythonapi is done as #define my_pythonapi(args, func) set_pythonapi_via_python(args, func) with the set_pythonapi_via_python function being defined in [wrapper.c](https://github.com/CCExtractor/ccextractor/blob/master/api/wrappers/wrapper.c#L8).
 * Thus, it can been observed that my_pythonapi takes two arguments when the compilation is done as extension module. In both the case, the first argument is struct ccx_s_options instance as used by api_add_param. But in case of compiling the extension module, the my_pythonapi function takes a second parameter which is the python callback function that CCExtractor would call when passing values from C to Python (a detailed discussion about this has been done later).
 * This function is not a mandatory function to call when using the CCExtractor binary.

### compile_params

Function declaration- **int compile_params(struct ccx_s_options *api_options,int argc)**

 * The compile_params function mainly compiles all the parameters supplied by the user and modifies the elements of the api_options on the basis of the parameters supplied by the user.
 * In this function, we add a dummy parameter ./ccextractor so that the parse_params function which is called from compile_params function properly compiles all the parameter except the first parameter as done in [here](https://github.com/CCExtractor/ccextractor/blob/master/src/lib_ccx/params.c#L1101).
 * This function then returns the return value as obtained by the [parse_params function](https://github.com/CCExtractor/ccextractor/blob/master/src/lib_ccx/params.c#L1098).

### call_from_python_api

Function declaration- **void call_from_python_api(struct ccx_s_options *api_options)**

 * The call_from_python_api function checks if the parameter -pythonapi was provided by the user. If the parameter was passed by the user then the global varable signal_python_api is set to 1 showing that the execution is done via Python modules; otherwise the value of signal_python_api is 0.
 * In case of clarifications, the user does not explicitly need to pass the parameter -pythonapi. It is passed by the my_pythonapi function and thus used by call_from_python_api to set the signal_python_api to 1.

### api_start

Function declaration- **int api_start(struct ccx_s_options api_options)**

 * This is the most important function of entire processing done by CCExtractor. After the entire compiling of parameters have been completed, then comes the stage when the actual processing is done.
 * The api_start is the function which is majorly responsible for extracting the caption frames and passing them back to Python for processing.

The user should note that the codeflow discussed above till this point
is generic to both CCExtractor binary as well as CCExtractor's Python
extension module. From this point onwards, the codeflow that has been
described is mainly how the Python extension module accepts the caption
frames via callback function and then processings done on the caption
frames to generate the output subtitle file (.srt) via Python.

 * The api_start function in case of CE-608 captions calls a function general_loop for processing of the sample(video) that needs to be processed which in turn makes a call to encode_sub which encodes the subtitle buffer obtained from the sample.
 * In [encode_sub function](https://github.com/CCExtractor/ccextractor/blob/master/src/lib_ccx/ccx_encoders_common.c#L1070), the sub_type is checked to be CE-608. If the sub_type is 608, then another check is made to check the value of [signal_python_api](https://github.com/CCExtractor/ccextractor/blob/master/src/lib_ccx/ccx_encoders_common.c#L1133). If the signal_python_api is set to 1, then a call to [pass_cc_buffer_to_python](https://github.com/CCExtractor/ccextractor/blob/master/src/lib_ccx/ccx_encoders_common.c#L1133) is made. Otherwise, the processing continues as if the call for processing was made from CCExtractor binary.

From the pass_cc_buffer_to_python function, the call is made to the
[extractor function](https://github.com/CCExtractor/ccextractor/blob/master/src/lib_ccx/ccx_encoders_python.c#L32),
then the extractor function in turns [calls the callback function](https://github.com/CCExtractor/ccextractor/blob/master/api/extractors/extractor.c#L88)
provided earlier via my_pythonapi function. The arguments given to the
callback function are the ones corresponding to the information content
of the caption frame which has been processed by CCExtractor. This
information is accessed via the Python SRT generator scripts which would
process the caption frames and write the processed information in the
output subtitle files. The following sections would be sequential
in-detail descriptions about how each process functions:

#### Python Encoder for CCExtractor

 * Following the architecture of CCExtractor’s codebase, a new file named ccx_encoders_python.c was added. The main reason of adding this file was to define the functions which would be called when the extraction process or CCExtractor extraction functionality is being performed via Python extension module. At this moment, since the extension module extends support only for CE-608 samples, only pass_cc_buffer_to_python function has been defined. Later on, when the binding’s support is extended to support other formats then in  that case other functions like pass_cc_bitmap_to_python and others would be included in this file following the architecture of other encoders.

### pass_cc_buffer_to_python

Function declaration- **int pass_cc_buffer_to_python(struct eia608_screen *data, struct encoder_ctx *context)**

 * This is the function where the actual work of passing the extracted caption buffer to Python extension modules for processing the caption frames is done.
 * The pass_cc_buffer_to_python function is called when the sample from which the caption frames are to be extracted is a CE-608 sample and the call for extraction is made from Python extension module. 
 * In this function, whenever a caption frame element is extracted, be it the srt_counter, caption timing information or any information related to the text, font or color grid of the CE-608 captions, then that information is passed to [extractor function](https://github.com/CCExtractor/ccextractor/blob/master/src/lib_ccx/ccx_encoders_python.c#L32) defined in extractors/ directory. A detailed description about how exactly the extractors function would be included in the next section.

#### Extractors for bindings

 * As documented in the previous section, when the extraction of CE-608 caption frames in done via Python, then the call is made to pass_cc_buffer_to_python function defined in ccx_encoders_python.c. In this function, after extracting lines in a caption frame (lines may belong to any of the text, font or color grid for CE-608), those lines are passed to [python_extract_g608_grid](https://github.com/CCExtractor/ccextractor/blob/master/api/extractors/extractor.c#L3) function defined in extractors.c. 

### python_extract_g608_grid

Function declaration- **void python_extract_g608_grid(unsigned h1, unsigned m1, unsigned s1, unsigned ms1, unsigned h2, unsigned m2, unsigned s2, unsigned ms2, char* buffer, int identifier, int srt_counter, int encoding)**

 * The main aim of using python_extract_g608_grid function is to able to identify the lines belonging to a particular frame and then passing these lines to the Python callback function with added identifiers for identification as to which CE-608 grid those lines belong to in a particular caption frame. More documentation about the identifiers and the nomenclature used for the bindings has been documented in the ‘Support for only CE-608 captions’ section and the user is advised to read that section to get a better understanding of the nomenclature.
 * The arguments passed to python_extract_g608_grid  include encoding which is the encoding that CCExtractor would have used to write the output subtitle file. Thus, the encoding is passed from CCExtractor to Python via the callback function so that the output subtitle file generated by Python would have the same encoding as the output generated by CCExtractor would have had.
 * Out of all the arguments that are passed to the python_extract_g608_grid function, the one interesting argument is the [identifier](https://github.com/CCExtractor/ccextractor/blob/master/api/extractors/extractor.c#L4) argument which has different values depending on the type of caption frame line it is called with. For example, if the line passed to python_extract_g608_grid function is a line belonging to its color grid, then the value of the identifier would be 2. Similarly, we have:
   * identifier = 0 -> adding start and end time 
   * identifier = 1 -> subtitle
   * identifier = 2 -> color
   * identifier = 3 -> font
 * This is how the python_extract_g608_grid function is able to generate the entire caption frame for a CE-608 sample along with timings.

#### Callback Function architecture

 * When using the extension module, when a particular C function is called from Python, the control is transferred to C and returned to Python only after the execution of the function. However, according to the adopted architecture, a single function would process the entire sample and extract all the caption frames until the control is passed back to Python for processing the captions in Python. Thereupon, for further processing in Python the user would have had to wait until the end of the extraction of all the caption frames from the sample. This would violate the basic ideology that the module should be able to process the caption frames in Python as they are extracted in CCExtractor rather than waiting till the end of extraction from the entire sample.
 * As a result of this, the callback function architecture was adopted. The main advantage of this architecture is that the moment a line from the caption frame is extracted the line is passed via a callback function to Python and the processing of the extracted line could be done in Python. 
 * In the present architecture, the user has a flexibility to tell CCExtractor which Python function would act as a callback function and a mechanism has been designed to convey this function to CCExtractor. This has been done with the use of my_pythonapi function as discussed in the previous sections.
 * NOTE: In the api_testing.py, I have defined the callback function to be named [callback](https://github.com/CCExtractor/ccextractor/blob/master/api/api_testing.py#L25). However, the user has complete freedom to define any name for the callback function. The user needs to note that the callback function would be getting nothing but a line from the caption frame that is extracted by CCExtractor. Further processing of the extracted line is the responsibility of the user.
 * After defining the callback function, the user needs to make sure that this function is passed via Python to CCExtractor so that it can be used for callback. For doing so, the user needs to set the second argument of the function my_pythonapi as the callback function. This has been done in the api_testing.py script and the user can refer to it for [example](https://github.com/CCExtractor/ccextractor/blob/master/api/api_testing.py#L16).
 * A detailed description about why a single line of the caption frame is passed via the callback function and not the entire frame is described in detail in later sections.
 * Also, when the user passes the callback function via Python to CCExtractor so the [my_pythonapi function](https://github.com/CCExtractor/ccextractor/blob/master/api/wrappers/wrapper.c#L10) saves a pointer to this function as an element to a global structure, array, defined and declared in ccextractor.h. The element [reporter](https://github.com/CCExtractor/ccextractor/blob/master/src/ccextractor.h#L37) holds the callback function passed by user via Python. 
 * Whenever the user wants to pass a line to the callback function then the user needs to call the function [run](https://github.com/CCExtractor/ccextractor/blob/master/src/ccextractor.c#L553) which has been defined in ccextractor.c.

### run

Function declaration- **void run(PyObject * reporter, char * line, int encoding)**

 * The run function takes two arguments and their description is as follows:
     * The first argument is the callback function which the user passes via Python. According to present architecture, this callback function is contained by the element reporter contained in the global structure named array. So the first argument is array.reporter.
     * The second argument to the run function is the line which needs to be passed to Python.

This is how the callback mechanism works for passing the lines from C to
Python in real time.

#### Processing output in Python

 * As described in the previous sections, the extension modules just return a single line from the caption frames. The processing of the caption frames to generate the output subtitle file is done in Python.
 * A script to generate an output subtitle file from the extracted captions frames in Python has been written. The api_testing.py has a function named callback which acts as a callback function returning the extracted caption lines in Python. These lines then are passed to [generated_output_srt](https://github.com/CCExtractor/ccextractor/blob/master/api/api_support.py#L7) in api_support.py described in the api/ directory. Thereupon, the function searches if the line has specific identifier which are used to decide how the output would be generated. A detailed section has been included in this documentation regarding the nomenclature used for processing different lines in CE-608 format caption fields (Support for only CE-608 captions section). The main reason for doing so is to avoid any buffering in C to hold the caption lines until the entire caption frames are extracted. This facilitates real time processing of the extracted caption frames.
 * For [getting the output filename](https://github.com/CCExtractor/ccextractor/blob/master/api/api_support.py#L10) from CCExtractor which would then be used to write the output srt file from Python, whenever the code is run from the extension module the first line that is passed via the callback function is the output filename generated by CCExtractor. This is incorporated by [calling the callback function from init_write](https://github.com/CCExtractor/ccextractor/blob/master/src/lib_ccx/output.c#L58) function defined in the src/lib_ccx/output.c file. The line passed to the callback function is of the format filename-<name of the output file to be generated> and this is then used to generate the output file. This line is then captured in the generate_output_srt function defined in the api_support.py.
 * However, if the user wants the flexibility of defining the filename in a different manner, then for such outputs, the user must make changes in the generate_output_srt function to set the filename and ignoring the first line that appears in Python via the callback function.

#### Support for only CE-608 captions

**For understanding the CE-608 caption format, the user is advised to refer to this [documentation on CE-608](https://github.com/CCExtractor/ccextractor/blob/master/docs/G608.TXT).**

 * The Python extension module is so far able to extract the captions frames from CE-608 samples. In samples with CE-608, the caption frames that are extracted by CCExtractor are in the form a 15x32 grid which depicts the screen. Thus, the information regarding the font of the captions, the colour they would be having on the screen as well as their alignment on the screen is captured in font,color and text grids respectively.
 * Using Python modules each of such grids can be accessed in Python. However, as described in the previous section the callback function gets a single line and not the entire grid from CCExtractor, some processing needs to be done in Python for getting the user required grids per caption frames.
 * The functions which would be acting as the processing and buffering functions for grid generations are present in the [ccx_to_python_g608.py](https://github.com/CCExtractor/ccextractor/blob/master/api/ccx_to_python_g608.py). The two major functions are [return_g608_grid](https://github.com/CCExtractor/ccextractor/blob/master/api/ccx_to_python_g608.py#L15) and [g608_grid_former](https://github.com/CCExtractor/ccextractor/blob/master/api/ccx_to_python_g608.py#L1). The g608_grid_former is mainly used to form the grid from lines obtained at the callback function.
 * The main advantage of the return_g608_grid function is that the user can generate whatever pattern the user desires to process in Python. For accessing various different combinations of the font, color and text grids in CE-608, a [help_string](https://github.com/CCExtractor/ccextractor/blob/master/api/ccx_to_python_g608.py#L17) has been defined in the return_g608_grid function in the ccx_to_python_g608.py file which describes on the value of mode to be passed to this function to get proper combination of the grids.
 * In the earlier sections it has been stated that the callback function in Python is not passed with the entire caption frame but just one single line from the frame, a particular nomenclature has been devised to make sure that the lines belonging to the same caption frames are identified in the Python interface. The nomenclature is as follows:
     * For every frame, the [first line](https://github.com/CCExtractor/ccextractor/blob/master/api/extractors/extractor.c#L88) that is passed to the callback function is the srt_counter which indicates the identifier value of the caption frame that would be extracted next.
     * Following the srt_counter, the next line would contain a conjunction of the [start time and end time](https://github.com/CCExtractor/ccextractor/blob/master/api/extractors/extractor.c#L96) of the caption frame with respect to the timings when the captions would be visible on the screen. The start_time and end_time would be conjuncted as start_time-<start time>t end_time-<end time>n and the user needs to process this line to get the timings. This processing in case of getting a srt file as an output has been done in the [generate_output_srt function](https://github.com/CCExtractor/ccextractor/blob/master/api/api_support.py#L18).
     * After the timings have been sent via the callback function, until the next srt_counter is extracted, the lines containing information about the color, font or text grids of CE-608 samples are passed via the callback  function to Python.
     * For processing the grids separately, the color grid could be identified by identifying the presence of color[<srt_counter value>]:<color grid line> in the line obtained from the callback function. Similarly, for the font and text grids, the nomenclatures are font[<srt_counter value>]:<font grid line> and text[<srt_counter value>]:<text grid line> respectively. Processing a grid on the basis of such a nomenclature has been done in the g608_grid_former in the ccx_to_python_g608.py file.
     * After the entire caption frame has been sent via the callback function to Python for further processing, when the extraction of present caption frames finishes and CCExtractor shifts to a new frame, then a line containing ***END OF FRAME*** is passed via the callback function from C to Python. The user needs to catch this line in order to get the signal that from the next line onwards a new caption frame would begin. [Similar approach](https://github.com/CCExtractor/ccextractor/blob/master/api/api_support.py#L28) has been implemented in the function generate_output_srt in the api_support.py file.

This is how the entire CE-608 is transmitted to Python and the user
needs to follow the nomenclature in order to get the caption frames in
Python.

 * However, if the user thinks to modify the nomenclature in accordance with some other nomenclature that suits their use case, then the user can do so by editing the [python_extract_g608_grid](https://github.com/CCExtractor/ccextractor/blob/master/api/extractors/extractor.c#L3) function in the extractor.c file. In this file, the user needs to find the lines where the function run is called with its first parameter being the callback function that is passed from Python and the second parameter being the line which is to be passed to Python.

#### Wrappers for the extension module

 * In case of using an API, it is highly desired to set the parameters desired by the user not via command line but as call to built-in functions. This gave rise to the necessity of wrapper functions which can be called to set certain parameters for directing the functioning of the bindings.
 * The wrappers have been defined in the [wrapper.c](https://github.com/CCExtractor/ccextractor/blob/master/api/wrappers/wrapper.c) file in api/wrappers/ directory. The user can use just call the wrappers to set some parameters. More wrappers can be defined according to the architecture followed in wrapper.c.
 * The user needs to note that the wrappers can be called anytime in between adding parameters to CCExtractor instance (as done in api_testing.py) and before calling the compile_params function from the CCExtractor module.
 * Another thing to note about the wrapper is that, the my_pythonapi wrapper function is a very important wrapper function. It tells CCExtractor that the call has been made using the Python module and thus the functioning of CCExtractor is altered. Hence, if the user intends to use the Python module the user is always advised to call this wrapper function with its first argument to be the object returned by api_init function from CCExtractor module and second argument being the callback function which would be called by the CCExtractor to pass the extracted caption lines back to Python.

#### Test Script

 * Once the Python module are generated then the user can use them by importing ccextractor module in Python. 
 * For testing the output of the bindings a test script, [api_testing.py](https://github.com/CCExtractor/ccextractor/blob/master/api/api_testing.py). But to mention, the module at this stage only supports generating a subtitle file from the CE-608 standard samples only.
 * Another testing feature, that has been added is that the user can use [recursive_tester.py](https://github.com/CCExtractor/ccextractor/blob/master/api/recursive_tester.py) to generate the subtitle files for all the samples from a directory. The only parameter needed to run this script is the location of all the samples.

#### Silent API

 * The Python bindings have been designed in such a way that the API is silent in itself as well as in the form of output generation. Silent in itself means that the API doesn’t write out any output to the STDOUT and the entire output of CCExtractor is silenced when the module is used for extraction of caption frames. This feature has been made possible by passing a parameter -pythonapi internally in api_testing.py using the function my_pythonapi() from the ccextractor module. The -pythonapi internally makes CCExtractor to silence all the outputs that could have been generated otherwise. 
 * If the user wants to add some print functionality from the CCExtractor, then may be defining the prints using printf C function could be an option. Note that the user cannot use the mprint function to get prints from the extension module from inside the CCExtractor C code part as used in CCExtractor to get the desired STDOUT prints as these are silenced via -pythonapi.

#### Work status

 * The proposal made by me for this project had a major component of multi-threading to let CCExtractor’s Python bindings run the CCExtractor’s extraction process in multi-threads.
 * However, the end goal was modified while the GSOC 2017 coding period and after Second Phase Evaluation, the main aim was to create a Python extension module for CCExtractor which could process CE-608 video samples, extract the caption information present in them and pass this information to Python for further processing. The module was expected to be silent and the output generation from the caption information present in the video sample has to be done via Python.
 * The present status of the extension module is that the module can extract caption information from CE-608 standard video samples and pass the caption information to Python. Further work has also been done to process this caption information to generate an output subtitle(srt) file (the user is advised to check completion of comparing_text_font_grids function sub-section under the future work section).

#### Future Work

### Identifying the input format and raising errors if unsupported

 * CCExtractor does not process any non-video files. Similarly, the processing of non-video files is not supported by extension module. However, since the API has been designed to be silent, the module doesn’t output any error log stating that the input file is a non-video file and it cannot be processed. 
 * This is a much desired feature and the present version of CCExtractor extension module lacks this feature. I would be working on this feature post GSOC 2017 but if any user finds that this feature has not been added until they start contribution to CCExtractor’s extension module, then their work on this feature would be highly appreciated.
 * For adding this feature to extension module, the extension module must be extended to process the return value from CCExtractor as done in the [api_start function](https://github.com/CCExtractor/ccextractor/blob/master/src/ccextractor.c#L71). When the sample (non-video) is processed via CCExtractor’s binary, then the processing is stopped by raising an ‘Invalid option to CCExtractor Library’ error. However, since the extension module has been designed to be silent, this error message is suppressed. Hence, the user should extend the test scripts to process the return value of api_start function in python extension module according to the constants defined in [ccx_common_common.h](https://github.com/CCExtractor/ccextractor/blob/master/src/lib_ccx/ccx_common_common.h).

### Callback class mechanism

 * The present architecture uses a callback mechanism to pass the extracted caption lines from the caption frames of CE-608 captions to Python for further processing. In the callback mechanism, a callback function is supplied to CCExtractor in C via the my_pythonapi function which stores the callback function as a PyObject* in the global variable array. However, according to Python documentation on C-API, everything in Python is a PyObject; be it a function, a tuple or a class.
 * So, the ideology is to replace the present callback function by a class which can have many methods that the user can use for different use cases.
 * An example of such an implementation has been done [here](https://github.com/Diptanshu8/ccextractor/blob/callback_class/api/api_testing.py#L27). The user needs to note that for accessing the Python class in C, some modifications need to be done to the run function defined in ccextractor.c and a sample example for calling a class method named ‘callback’ could be found [here](https://github.com/Diptanshu8/ccextractor/blob/callback_class/src/ccextractor.c#L553).
 * Also, an important point to be noted in this case is that the user needs to pass the callback function’s name to run function in C so that the corresponding callback method of the class passed via my_pythonapi could be called via C. As an example, the callback method’s name has been provided [here](https://github.com/Diptanshu8/ccextractor/blob/callback_class/src/ccextractor.c#L562).
 * For understanding the exact implementation of this approach, I would recommend the user to understand C-API for Python as the documentation is quite extensive to every use case.

### Completion of comparing_text_font_grids function

 * The Python extension module for CCExtractor is able to pass lines of the caption frames for different grids of CE-608 captions. However, for generating the subtitle file from the caption grids, the text grid needs to be modified according to the color grid as well as font grid. In CCExtractor, this job is done at the function, [get_decoder_line_encoded](https://github.com/Diptanshu8/ccextractor/blob/callback_class/src/lib_ccx/ccx_encoders_helpers.c#L234).
 * For generation of subtitle files (.srt files) from Python, an equivalent version of get_decoder_line_encoded has been implemented in Python and has been defined as [comparing_text_font_grids](https://github.com/CCExtractor/ccextractor/blob/master/api/python_srt_generator.py#L56) in python_srt_generator.py
 * However, as the user can note that this function is not a complete implementation of get_decoder_line_encoded function, completion of this function’s definition is a matter of future work.

### Adding more wrapper functions

 * As described in the ‘Wrappers for the extension module’ section, more wrapper functions are needed to be declared in the [wrapper.c](https://github.com/CCExtractor/ccextractor/blob/master/api/wrappers/wrapper.c) file. For example, few wrappers have been defined. More wrapper functions can be defined in a similar manner.

### Extending the module to support other caption formats

 * In this version, CCExtractor’s extension module supports processing of video samples having CE-608 standard captions in them and writing these captions to output subtitle (.srt) files.
 * However, CCExtractor in itself has support for other caption standards like DVB, 708 etc. So, extension of module to extract of caption information from samples containing the caption information in these formats is a future task.
 * The user should note that the information passed from CE-608 to Python is in raw form as lines which are then used to form the 608 grids. Similarly, the extension to other formats must consider passing the raw information of caption in respective format and then processing the information extracted by CCExtractor in Python.
 * While extending, the architecture to be followed for ccx_encoders_python should be consistent to other encoders in the codebase to maintain uniformity. Thus for DVB samples, a function name pass_cc_bitmap_to_python and for 708 samples pass_cc_subtitle_to_python need to be declared in ccx_encoders_python.c.
