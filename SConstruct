import os, sys, platform
def bitscanrev(x, c=-1):
    return bitscanrev(x/2, c+1) if x else c

env = Environment()
#print env['TOOLS']
AddOption('--debugbuild', dest='debug', nargs='?', const=True, help='enable debug build')
AddOption('--debugprint', dest='debugprint', nargs='?', const=True, help='enable lots of debug printing (windows only)')
AddOption('--fullsanitychecks', dest='fullsanitychecks', nargs='?', const=True, help='enable full sanity checking on every memory op')
AddOption('--force32', dest='force32', help='force 32 bit build on 64 bit machine')
AddOption('--sse', dest='sse', nargs=1, type='int', default=1, help='set SSE used (0-4) on 32 bit x86. Defaults to 1 (SSE1).')
AddOption('--replacesystemallocator', dest='replacesystemallocator', nargs='?', const=True, help='replace all usage of the system allocator in the process when loaded')
AddOption('--tolerant', dest='tolerant', nargs=1, default=1, help='enable tolerance of the system allocator (not guaranteed). On by default.')
AddOption('--magicheaders', dest='magicheaders', nargs='?', const=True, help='enable magic headers (guaranteed tolerance of the system allocator)')
AddOption('--useallocator', dest='useallocator', nargs=1, type='int', default=1, help='which allocator to use')
AddOption('--uselocks', dest='uselocks', nargs=1, type='int', default=1, help='which form of locking to use')
AddOption('--nospinlocks', dest='nospinlocks', nargs='?', const=True, help='use system mutexs rather than CPU spinlocks')
AddOption('--largepages', dest='largepages', nargs='?', const=True, help='enable large page support')
AddOption('--fastheapdetection', dest='fastheapdetection', nargs='?', const=True, help='enable fast system-specific heap detection')
AddOption('--maxthreadsinpool', dest='maxthreadsinpool', nargs=1, type='int', help='sets how much memory bloating to cause for more performance')
AddOption('--defaultgranularity', dest='defaultgranularity', nargs=1, type='int', help='sets how much memory to claim or release from the system at one time')
AddOption('--threadcachemax', dest='threadcachemax', nargs=1, type='string', help='sets what allocations should use the threadcache')
AddOption('--threadcachemaxbins', dest='threadcachemaxbins', nargs=1, type='int', help='sets the threadcache binning')
AddOption('--threadcachemaxfreespace', dest='threadcachemaxfreespace', nargs=1, type='int', help='sets when the threadcache should be garbage collected')

# Force scons to always use absolute paths in everything (helps debuggers to find source files)
env['CCCOM']   =    env['CCCOM'].replace('$CHANGED_SOURCES','$SOURCES.abspath')
env['SHCCCOM'] =  env['SHCCCOM'].replace('$CHANGED_SOURCES','$SOURCES.abspath')
env['CXXCOM']  =   env['CXXCOM'].replace('$CHANGED_SOURCES','$SOURCES.abspath')
env['SHCXXCOM']= env['SHCXXCOM'].replace('$CHANGED_SOURCES','$SOURCES.abspath')
architecture="generic"
env['CPPDEFINES']=[]
env['CCFLAGS']=[]
env['LIBS']=[]
env['LINKFLAGS']=[]
if env.GetOption('debugprint'): env['CPPDEFINES']+=["USE_DEBUGGER_OUTPUT"]
if env.GetOption('fullsanitychecks'): env['CPPDEFINES']+=["FULLSANITYCHECKS"]
if env.GetOption('replacesystemallocator'): env['CPPDEFINES']+=["REPLACE_SYSTEM_ALLOCATOR"]
if env.GetOption('tolerant'): env['CPPDEFINES']+=["ENABLE_TOLERANT_NEDMALLOC"]
if env.GetOption('magicheaders'): env['CPPDEFINES']+=["USE_MAGIC_HEADERS"]
env['CPPDEFINES']+=[("USE_ALLOCATOR",env.GetOption('useallocator'))]
env['CPPDEFINES']+=[("USE_LOCKS",env.GetOption('uselocks'))]
if env.GetOption('nospinlocks'): env['CPPDEFINES']+=[("USE_SPIN_LOCKS",0)]
if env.GetOption('largepages'): env['CPPDEFINES']+=["ENABLE_LARGE_PAGES"]
if env.GetOption('fastheapdetection'): env['CPPDEFINES']+=["ENABLE_FAST_HEAP_DETECTION"]
if env.GetOption('maxthreadsinpool'): env['CPPDEFINES']+=[("MAXTHREADSINPOOL",env.GetOption('maxthreadsinpool'))]
if env.GetOption('defaultgranularity'): env['CPPDEFINES']+=[("DEFAULT_GRANULARITY",env.GetOption('defaultgranularity'))]
if env.GetOption('threadcachemax'):
    threadcachemax=int(env.GetOption('threadcachemax'))
    if threadcachemax<=32: threadcachemax=0
    env['CPPDEFINES']+=[("THREADCACHEMAX",threadcachemax)]
    if not env.GetOption('threadcachemaxbins') and threadcachemax:
        maxbins=bitscanrev(threadcachemax)-4;
        print "THREADCACHEMAX set but not THREADCACHEMAXBINS, so auto-setting THREADCACHEMAXBINS =", maxbins
        env['CPPDEFINES']+=[("THREADCACHEMAXBINS",maxbins)]
if env.GetOption('threadcachemaxbins'): env['CPPDEFINES']+=[("THREADCACHEMAXBINS",env.GetOption('threadcachemaxbins'))]
if env.GetOption('threadcachemaxfreespace'): env['CPPDEFINES']+=[("THREADCACHEMAXFREESPACE",env.GetOption('threadcachemaxfreespace'))]

# Am I in a 32 or 64 bit environment? Note that not specifying --sse doesn't set any x86 or x64 specific options
# so it's good to go for ANY platform
if sys.platform=="win32":
    # Even the very latest scons still screws this up :(
    env['ENV']['INCLUDE']=os.environ['INCLUDE']
    env['ENV']['LIB']=os.environ['LIB']
    env['ENV']['PATH']=os.environ['PATH']
    if not env.GetOption('force32') and os.environ.has_key('LIBPATH') and -1!=os.environ['LIBPATH'].find("\\amd64"):
        architecture="x64"
    else:
        architecture="x86"
        if   env.GetOption('sse')==1: env['CCFLAGS']+=[ "/arch:SSE" ]
        elif env.GetOption('sse')>=2: env['CCFLAGS']+=[ "/arch:SSE2" ]
        if   env.GetOption('sse')>=3: env['CPPDEFINES']+=[("__SSE3__", 1)]
        if   env.GetOption('sse')>=4: env['CPPDEFINES']+=[("__SSE4__", 1)]
else:
    if not env.GetOption('force32') and ('x64' in platform.machine() or 'x86_64' in platform.machine()):
        architecture="x64"
    elif platform.machine() in ['i386', 'i486', 'i586', 'i686']:
        architecture="x86"
        if env.GetOption('sse'):
            env['CCFLAGS']+=["-mfpmath=sse"]
            if env.GetOption('sse')>1: env['CCFLAGS']+=["-msse%s" % str(env.GetOption('sse'))]
            else: env['CCFLAGS']+=["-msse"]

# Am I building a debug or release build?
if env.GetOption('debug'):
    env['CPPDEFINES']+=["DEBUG", "_DEBUG"]
    variant=architecture+"/Debug"
else:
    env['CPPDEFINES']+=["NDEBUG"]
    variant=architecture+"/Release"

# Am I building for Windows or POSIX?
if sys.platform=='win32':
    env['CPPDEFINES']+=["WIN32", "_WINDOWS", "UNICODE", "_UNICODE"]
    env['CCFLAGS']+=["/GF"]             # Eliminate duplicate strings
    env['CCFLAGS']+=["/Gy"]             # Seperate COMDATs
    env['CCFLAGS']+=["/Zi"]             # Program database debug info
    if env.GetOption('debug'):
        env['CCFLAGS']+=["/Od", "/MDd"]
    else:
        env['CCFLAGS']+=["/O2", "/MD"]
        #env['CCFLAGS']+=["/GL"]         # Do link time code generation
    env['LIBS']+=["psapi", "user32", "advapi32"]
    env['LINKFLAGS']+=["/DEBUG"]                # Output debug symbols
    env['LINKFLAGS']+=["/LARGEADDRESSAWARE"]    # Works past 2Gb
    env['LINKFLAGS']+=["/DYNAMICBASE"]          # Doesn't mind being randomly placed
    env['LINKFLAGS']+=["/NXCOMPAT"]             # Likes no execute

    env['LINKFLAGS']+=["/ENTRY:DllPreMainCRTStartup"]
    env['LINKFLAGS']+=["/VERSION:1.1.0"]        # Version
    env['LINKFLAGS']+=["/MANIFEST"]             # Be UAC compatible
    
    if not env.GetOption('debug'):
        env['LINKFLAGS']+=["/OPT:REF", "/OPT:ICF"]  # Eliminate redundants
        #env['LINKFLAGS']+=["/PGD:${VARIANT}/nedmalloc.pgd"]
        #env['LINKFLAGS']+=["/LTCG:PGINSTRUMENT"]
        #env['LINKFLAGS']+=["/LTCG:PGUPDATE"]
else:
    env['CPPDEFINES']+=[]
    env['CCFLAGS']+=["-Wall"]
    if env.GetOption('debug'):
        env['CCFLAGS']+=["-O0", "-g"]
    else:
        env['CCFLAGS']+=["-O2", "-g"]
    if env.GetOption('uselocks'):
        env['LIBS']+=["pthread"]
    env['LINKFLAGS']+=[]

# Build
env['VARIANT']=variant
nedmalloclib=SConscript("SConscript", variant_dir=variant, duplicate=False, exports="env")
Return("nedmalloclib")
