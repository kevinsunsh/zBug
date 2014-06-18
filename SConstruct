#
# Copyright 2010-2014 Fabric Software Inc. All rights reserved.
#

import os, sys

useNinja = True

if not os.environ.has_key('FABRIC_BUILD_OS'):
  print "Must source fabric-build-env.sh first"
  sys.exit(1)

if not os.path.exists('llvm'):
  print "No 'llvm' folder found!"
  sys.exit(1)

env = Environment(
  ENV = {
    'PATH': os.environ['PATH'],
  },
  # TARGET_ARCH must be set when the Environment() is created in order
  # to pull in correct x86 vs x64 paths on Windows
  TARGET_ARCH = os.environ['FABRIC_BUILD_ARCH'],
  )

llvmSourceDir = env.Dir('#llvm')
buildDir = env.Dir('#build')
llvmBuildDir = buildDir.Dir('llvm')

llvmConfigCmd = [
  'cd', str(llvmBuildDir), '&&'
  'cmake',
  '-DCMAKE_BUILD_TYPE:STRING=' + os.environ['FABRIC_BUILD_TYPE'],
  '-DLLVM_INCLUDE_TESTS:BOOL=OFF',
  '-DLLVM_INCLUDE_EXAMPLES:BOOL=OFF',
  '-DLLVM_ENABLE_TERMINFO:BOOL=OFF',
  ]

if os.environ['FABRIC_BUILD_OS'] == 'Darwin':
  llvmFlags = '-mmacosx-version-min=10.7 -stdlib=libstdc++ -fvisibility=hidden'
  llvmConfigCmd += [
    '-DCMAKE_C_FLAGS:STRING=' + llvmFlags,
    '-DCMAKE_C_LINK_FLAGS:STRING=' + llvmFlags,
    '-DCMAKE_CXX_FLAGS:STRING=' + llvmFlags,
    '-DCMAKE_CXX_LINK_FLAGS:STRING=' + llvmFlags,
    ]
elif os.environ['FABRIC_BUILD_OS'] == 'Linux':
  llvmFlags = '-fPIC'
  llvmConfigCmd += [
    '-DCMAKE_EXE_LINKER_FLAGS:STRING=\'-static-libstdc++ -static-libgcc\'',
    '-DCMAKE_SHARED_LINKER_FLAGS:STRING=\'-static-libstdc++ -static-libgcc\'',
    '-DCMAKE_MODULE_LINKER_FLAGS:STRING=\'-static-libstdc++ -static-libgcc\'',
    ]
  llvmConfigCmd += [
    '-DLLVM_ENABLE_PIC:BOOL=ON',
    '-DCMAKE_C_FLAGS:STRING=' + llvmFlags,
    '-DCMAKE_CXX_FLAGS:STRING=' + llvmFlags,
    ]
  if os.environ['FABRIC_BUILD_DIST'] == 'CentOS':
    llvmConfigCmd += [
      '-DCMAKE_SIZEOF_VOID_P=8',
      '-DCMAKE_C_COMPILER:STRING=/opt/gcc-4.8/bin/gcc',
      '-DCMAKE_CXX_COMPILER:STRING=/opt/gcc-4.8/bin/g++',
    ]
elif os.environ['FABRIC_BUILD_OS'] == 'Windows':
  llvmFlags = '-D_ITERATOR_DEBUG_LEVEL=0 -D_SECURE_SCL=0'
  llvmConfigCmd += [
    '-DLLVM_USE_CRT_DEBUG:STRING=MTd',
    '-DLLVM_USE_CRT_RELEASE:STRING=MT',
    '-DLLVM_USE_CRT_RELWITHDEBINFO:STRING=MT',
    '-DCMAKE_C_FLAGS:STRING=' + llvmFlags,
    '-DCMAKE_CXX_FLAGS:STRING=' + llvmFlags,
    ]

if os.environ['FABRIC_BUILD_OS'] == 'Windows':
  llvmCMakeGenerator = "Visual Studio 10 Win64"
else:
  if useNinja:
    llvmCMakeGenerator = 'Ninja'
  else:
    llvmCMakeGenerator = 'Unix Makefiles'
  llvmConfigCmd += [
    '-G', llvmCMakeGenerator,
    llvmSourceDir.abspath
    ]

if os.environ['FABRIC_BUILD_OS'] == 'Windows':
  llvmVSArch = "x64"
  llvmBuildCmd = [
    'cd', str(llvmBuildDir), '&&'
    'devenv', 'LLVM.sln', '/build', '"' + '|'.join([os.environ['FABRIC_BUILD_TYPE'], llvmVSArch]) + '"'
    ]
else:
  if useNinja:
    llvmBuildCmd = [
      'ninja', '-C', str(llvmBuildDir), '-j' + str(GetOption('num_jobs'))
      ]
  else:
    llvmBuildCmd = [
      'make', '-C', str(llvmBuildDir), '-j' + str(GetOption('num_jobs'))
      ]

buildLLVM = env.AlwaysBuild(
  env.Command(
    buildDir,
    [llvmBuildDir],
    [
      Mkdir(llvmBuildDir),
      llvmConfigCmd,
      llvmBuildCmd,
    ],
  )
)

zBugVersion = "1.1"
zipBaseName = '-'.join([
  "FabricEngine-Debugger-zBug",
  zBugVersion,
  os.environ['FABRIC_BUILD_DIST'],
  "Python",
  os.environ['PYTHON_VERSION']
  ])
stageDir = env.Dir('dist').Dir(zipBaseName)
zipFiles = []

cpzBug = env.Install(stageDir, env.File('zBug'))
zipFiles.append(cpzBug)

if os.environ['FABRIC_BUILD_DIST'] == 'CentOS':
  lldbPythonDir = llvmBuildDir.Dir('lib64').Dir('python'+os.environ['PYTHON_VERSION'])
else:
  lldbPythonDir = llvmBuildDir.Dir('lib').Dir('python'+os.environ['PYTHON_VERSION'])

mklib =  env.Command(
  stageDir.Dir('lib'),
  llvmBuildDir.Dir('lib').File('liblldb.so'),
  [['mkdir', stageDir.Dir('lib')], ['cp', '-a', "$SOURCE"+"*", "$TARGET"]]
)
env.Depends(mklib, cpzBug)
zipFiles.append(mklib)

mkbin = env.Command(
  stageDir.Dir('bin'),
  llvmBuildDir.Dir('bin').File('lldb'),
  [['mkdir', stageDir.Dir('bin')], ['cp', '-a', "$SOURCE"+"*", "$TARGET"]]
)
env.Depends(mkbin, cpzBug)
zipFiles.append(mkbin)

cppython = env.Command(
  stageDir.Dir('python'),
  lldbPythonDir,
  [['cp', '-a', "$SOURCE", stageDir.Dir('lib')]]
)
env.Depends(cppython, mklib)
zipFiles.append(cppython)

lnpython =env.Command(
  stageDir.Dir('lldb'),
  lldbPythonDir.Dir('site-packages').Dir('lldb'),
  [['ln', '-sf', "./lib/python"+os.environ['PYTHON_VERSION']+"/site-packages/lldb", "$TARGET"]]
)
env.Depends(lnpython, cppython)
zipFiles.append(lnpython)

env.Depends(zipFiles, buildLLVM)

zipFile = env.Command(
  env.Dir('dist').File(zipBaseName + '.tar.bz2'),
  zipFiles,
  [['tar', '-C', 'dist', '-cjf', '$TARGET', zipBaseName]]
)

Alias('all', zipFile)

