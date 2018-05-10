Objective
------------
Executing, documenting and validating test cases can be a challenge for most customers; Robot Framework can help speed up this process. This is an example to deploy configurations and validate services on Arista switches using keywords in [AristaLibrary](https://github.com/aristanetworks/robotframework-aristalibrary).

Getting Started
------------
Install the [robotframework](http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#installation-instructions), [robotframework-aristalibrary](https://github.com/aristanetworks/robotframework-aristalibrary) and the [robotframework-sshlibrary](http://robotframework.org/SSHLibrary/#installation) as shown below.

```
pip install robotframework
pip install robotframework-aristalibrary
pip install robotframework-sshlibrary
```

Topology and Setup
------------

The topology as shown in the [diagram](https://github.com/arista-eosext/robot-examples/blob/master/mlag-varp-validation/topology.jpg) is simple with spines in MLAG and running VARP. VARP is configured for VLAN 100, the leafs have VLAN 100 SVIs and their default-gateway points to the VARP IP on spines. the device configs can be found in the "device-configs" directory for reference.


Test Suites
------------
The default configurations will have MLAG shutdown on spine_B and VLAN 100 SVI shutdown on leaf_B.

1. **validate.txt** : This test suite runs the following test cases:
   * MLAG verification on spines:

     Here the keyword "Get Command Output" is used to grab 'show mlag' output in JSON format and the Expect keyword is used to perform validation. You should expect this test case to fail as MLAG is shutdown on leaf_B.
   * VARP verification on spines:

     Here the keyword "Run Cmds" is used to grab 'show ip virtual-router' output in JSON format and the built-in keyword "Should Be Equal" is used to perform validation. You should expect this test case to pass as VARP is active on both switches.
   * ICMP check to VARP gateways:

     Here the "Run Cmds" keyword is used to run pings from the leafs to the VARP gw on spines. We then use the built-in keyword "Should Match Regexp" to run a match on packet percentage loss. You should expect this test case to pass when the ping is initiated from leaf_A but should fail from leaf_B as SVI is shutdown on leaf_B.

2. **deploy-and-validate.txt** : This test suite first runs configuration commands to enable MLAG on spine_B and VLAN 100 SVI  on leaf_B. After running the configuration commands, the test suite re-runs the test cases from the first test suite, validate.txt, but this time you should expect all test cases to pass.

Running The Test Suites
------------

```
pybot --pythonpath=AristaLibrary --variablefile myvars.yaml test-suites/validate.txt
pybot --pythonpath=AristaLibrary --variablefile myvars.yaml test-suites/deploy-and-validate.txt
```

**Executing validate.txt**

As you can see below the MLAG test cases failed, and the ping from leaf-B to it's gateway failed.

```
sureshk:robot-demo sureshk$ pybot --pythonpath=AristaLibrary --variablefile myvars.yaml test-suites/validate.txt
==============================================================================
Validate :: A simple Robot Framework test suite to Validate Services
==============================================================================
Check MLAG Status on spine-A                                          | FAIL |
AristaLibrary.Expect: Key: '[u'state']', Found: 'inactive', Expected: 'active'
------------------------------------------------------------------------------
Check MLAG Status on spine-B                                          | FAIL |
AristaLibrary.Expect: Key: '[u'state']', Found: 'disabled', Expected: 'active'
------------------------------------------------------------------------------
Check VARP Status on spine-A                                          | PASS |
------------------------------------------------------------------------------
Check VARP Status on spine-B                                          | PASS |
------------------------------------------------------------------------------
Test if ping to IPv4 VARP Gateway passes from leaf-A                  | PASS |
------------------------------------------------------------------------------
Test if ping to IPv4 VARP Gateway passes from leaf-B                  | FAIL |
'connect: Network is unreachable
' does not match '(\d+)% packet loss'
------------------------------------------------------------------------------
Validate :: A simple Robot Framework test suite to Validate Services  | FAIL |
6 critical tests, 3 passed, 3 failed
6 tests total, 3 passed, 3 failed
==============================================================================
Output:  /Users/sureshk/EOS_Certifications/robot-demo/output.xml
Log:     /Users/sureshk/EOS_Certifications/robot-demo/log.html
Report:  /Users/sureshk/EOS_Certifications/robot-demo/report.html
sureshk:robot-demo sureshk$
```
**Executing deploy-and-validate.txt**

All test cases should pass as expected.

```
sureshk:robot-demo sureshk$ pybot --pythonpath=AristaLibrary --variablefile myvars.yaml test-suites/deploy-and-validate.txt
==============================================================================
Deploy-And-Validate :: A simple Robot Framework to deploy MLAG and VARP
==============================================================================
Enable MLAG on spine-B                                                | PASS |
------------------------------------------------------------------------------
Enable VLAN 100 SVI on leaf-B                                         | PASS |
------------------------------------------------------------------------------
Pause few seconds for MLAG to sync                                    | PASS |
------------------------------------------------------------------------------
Check MLAG Status on spine-A                                          | PASS |
------------------------------------------------------------------------------
Check MLAG Status on spine-B                                          | PASS |
------------------------------------------------------------------------------
Check VARP Status on spine-A                                          | PASS |
------------------------------------------------------------------------------
Check VARP Status on spine-B                                          | PASS |
------------------------------------------------------------------------------
Test if ping to IPv4 VARP Gateway passes from leaf-A                  | PASS |
------------------------------------------------------------------------------
Test if ping to IPv4 VARP Gateway passes from leaf-B                  | PASS |
------------------------------------------------------------------------------
Deploy-And-Validate :: A simple Robot Framework to deploy MLAG and... | PASS |
9 critical tests, 9 passed, 0 failed
9 tests total, 9 passed, 0 failed
==============================================================================
Output:  /Users/sureshk/EOS_Certifications/robot-demo/output.xml
Log:     /Users/sureshk/EOS_Certifications/robot-demo/log.html
Report:  /Users/sureshk/EOS_Certifications/robot-demo/report.html
sureshk:robot-demo sureshk$
```

**Reports**

Each execution of a test suite in Robot Framework produces a log html file that gives a drill-down analysis of the test cases run, a report html file that gives a summary of the test cases run, and an xml output file.

References
------------
* [Official Robot Framework User Guide](http://robotframework.org/robotframework/#user-guide)
* [Arista Library for Robot Framework](https://github.com/aristanetworks/robotframework-aristalibrary)
* [Arista Library Keyword Repository](https://aristanetworks.github.io/robotframework-aristalibrary)
