# FixPrime

FixPrime is a project (still under development), whose scope is to provide a very flexible framework to develop applications based in the fix protocol (Financial Information Exchange). 
###Currently available features in test:
 - Full customization of the specification of all fix protocol versions.
 - Message parser.
 - Message serializer.
 - Unlimited Nesting of repeating groups.

###Complete set of features planned in the project scope:
  - Full protocols customization.
  - Message parser.
  - Message serializer.
  - Customization of callback for Fix protocol events.
  - Automatic message sequencing.
  - Automatic session layer interaction.
  - Full message's hierarchies navigation.
  - Unlimited Nesting of repeating groups.
  - Automatic validation of Fix protocol datatypes.
  - Automatic validation of field's valid values.
  - Definition of multiple session engines
  - Behavior as Acceptor or initiator.
 
###Project's dependencies
[boost libraries] - Release 1.58.0
Download and install the boost libraries following the [boost libraries]'s site instructions.

[Jsoncpp] - Even though the project uses jsoncpp, you don't need to do anything about it, because the project has included the amalgamated version of jsoncpp source code.

###Quick Start using Fix Prime
Download and install the boost libraries following the [boost libraries]'s site instructions.

Clone the project in your system.
``` sh
git clone https://github.com/xjrueda/fixprime.git
```
Build the project using the MakeFile included.
```sh
make
```
copy the library into your local lib folder. Replace 'yourpath' for the appropiate path in your system.
```sh
sudo cp yourpath/fixprime/dist/Debug/GNU-Linux-x86/libsFixPrime.so /usr/local/lib
```
Start using the library.  Create a new project and link it with the libsFixPrime.so library
``` c++  
#include "FixDictionary.h"
using namespace std;
using namespace fprime;
int main(int argc, char** argv) {
    try {
        /* 
         * Instantiate  the fix dictionary.
         * You can use a shared pointer to the dictionary and load all protocol 
         * specifications only once in order to be used  in all application 
         * contexts.
         * 
         * Other option is instantiate directly a instance of the dictionary 
         * like this:
         *  
         * FixDictionary myFixDictionary;
         */
        FixDictionary::FixDictionaryPtr fixDictionaryPtr(new FixDictionary);
        /*
         * Load the protocols defined on the folder FixSpecifications.
         * The file FixVersion contains the path for every fix protocol specification
         * Replace yourpath for your path in your operating system
         */
        fixDictionaryPtr->loadProtocols("yourpath/FixPrime/FixSpecifications/FixVersions.json");
        /*
         * Use the desired protocol version
         */
        Protocol::ProtocolPtr protocolPtr = fixDictionaryPtr->getProtocol("FIX.4.4");
        cout << "Protocols loaded successfully." << endl;
        return 0;
    } catch (const exception& e) {
        cout << "An exception was raised loading fix protocols: " << e.what() << endl;
    }
```
### Producing messages

``` c++  
#include "FixDictionary.h"
using namespace std;
using namespace fprime;
int main(int argc, char** argv) {
    try {
        FixDictionary::FixDictionaryPtr fixDictionaryPtr(new FixDictionary);
        fixDictionaryPtr->loadProtocols("yourpath/FixPrime/FixSpecifications/FixVersions.json");
        Protocol::ProtocolPtr protocolPtr = fixDictionaryPtr->getProtocol("FIX.4.4");
        cout << "Protocols loaded sucessfully." << endl;
      
        //Get the desired message specifications
        Message logon = protocolPtr->getMessage("A");
      
        // Set the message values
        logon.getHeader()(8).setValue("FIX.4.4");
        logon.getHeader()(49).setValue("ANYSENDER");
        logon.getHeader()(56).setValue("ANYTARGET");
        logon.getBody()(553).setValue("myUserName");
        logon.getBody()(554).setValue("myPassword");
        
        // Serialize the message
        cout << "Logon message in fix format is:  " << logon.toFix() << endl;
        return 0;
    } catch (const exception& e) {
        cout << "An exception was raised loading fix protocols: " << e.what() << endl;
    }
}
```
### Populating repeating groups
``` c++  
#include "FixDictionary.h"
using namespace std;
using namespace fprime;
int main(int argc, char** argv) {
    try {
        FixDictionary::FixDictionaryPtr fixDictionaryPtr(new FixDictionary);
        fixDictionaryPtr->loadProtocols("yourpath/FixPrime/FixSpecifications/FixVersions.json");
        Protocol::ProtocolPtr protocolPtr = fixDictionaryPtr->getProtocol("FIX.4.4");
        cout << "Protocols loaded successfully." << endl;
      
        //Get the desired message specifications
        Message logon = protocolPtr->getMessage("A");
      
        // Set the message values
        newOrderSingle.getHeader()(8).setValue("FIX.4.4");
        newOrderSingle.getHeader()(49).setValue("ANYSENDER");
        newOrderSingle.getHeader()(56).setValue("ANYTARGET");
        newOrderSingle.getBody()(453).setValue("1");
        newOrderSingle.getBody()(453).appendGroupInstance();
        newOrderSingle.getBody()(453)[1](448).setValue("EXEFIRM00001");
        newOrderSingle.getBody()(453)[1](447).setValue("C");
        newOrderSingle.getBody()(453)[1](452).setValue("1");
        newOrderSingle.getBody()(453).appendGroupInstance();
        newOrderSingle.getBody()(453)[2](448).setValue("ENTERINFIRM01");
        newOrderSingle.getBody()(453)[2](447).setValue("C");
        newOrderSingle.getBody()(453)[2](452).setValue("7");
        
        
        // Serialize the message
        cout << "New Order message in fix format is:  " << newOrderSingle.toFix() << endl;
        return 0;
    } catch (const exception& e) {
        cout << "An exception was raised loading fix protocols: " << e.what() << endl;
    }
}
```
### Parsing raw messages
``` c++  
#include "FixParser.h"
#include "Message.h"
#include "Protocol.h"
#include "FixDictionary.h"
using namespace std;
using namespace fprime;
int main(int argc, char** argv) {
    try {
        FixDictionary::FixDictionaryPtr fixDictionaryPtr(new FixDictionary);
        fixDictionaryPtr->loadProtocols("yourpath/FixPrime/FixSpecifications/FixVersions.json");
        Protocol::ProtocolPtr protocolPtr = fixDictionaryPtr->getProtocol("FIX.4.4");
        string rawMessage = string("8=FIX.4.49=10335=A34=149=SENDER50=123X05") +
                            string("52=20150612-17:13:07.77856=TARGET98=0108=10141=Y") +
                            string("553=myuser554=mypwd10=189");
        // Instanciate the parser        
        FixParser parser;
        //assign the de desired protocol to the parser
        parser.setProtocol(protocolPtr);
        // fast parsing. It is just a tag=value map.
        FixParser::FlatMessage flatMessage = parser.explode(rawMessage);
        //Complete message parsing. Returns a Message Object of the corresponding fix message type
        Message fixmsg = parser.parseMessage(flatMessage);
        // get message values
        string msgType = fixmsg.getHeader()(35).getValue();
        cout << "Message type is " << msgType << endl;
        return 0;
    } catch (const exception& e) {
        cout << "An exception was raised loading fix protocols: " << e.what() << endl;
    }
}
```

New Features will be released soon.


[Boost libraries]:http://www.boost.org
[Jsoncpp]:https://github.com/open-source-parsers/jsoncpp