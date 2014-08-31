## nginx-vod-module tests

the accompanied python scripts test the nginx-vod-module, each test script has an associated template file
that contains environment specific parameters. the template file should be copied and edited to match the 
specific environment on which the tests are run.
the tests assume that nginx is running with the accompanied nginx.conf file.

### memory pollution

for additional coverage it is recommended to patch ngx_alloc & ngx_memalign (in src/os/unix/ngx_alloc.c)
so that they memset the allocated pointer with some value. the purpose of this change is to ensure the
code does not make any assumptions on allocated pointers being zeroed. ngx_calloc must not be patched.
to apply the patch - edit ngx_alloc and the two implementations of ngx_memalign, and add the following
block after the if:
    else {
        memset(p, 0xBD, size);
    }

### main.py

sanity + coverage tests, e.g.:
  1. bad request parameters
  2. bad upstream responses
  3. file not found

in order to run the tests nginx should be compiled with the --with-debug switch
  
### hls_compare.py

compares the nginx-vod hls implementation to some reference implementation

### validate_iframes.py

verifies using ffprobe that the byte ranges returned in an EXT-X-I-FRAMES-ONLY m3u8 indeed represent iframes

### buffer_cache

this folder contains a stress test for the buffer cache module. in order to execute the test, run:
 * NGX_ROOT=/path/to/nginx/sources
 * VOD_ROOT=/path/to/nginx/vod
 * cc -Wall $NGX_ROOT/src/core/ngx_palloc.c $NGX_ROOT/src/os/unix/ngx_alloc.c $NGX_ROOT/src/core/ngx_string.c $NGX_ROOT/src/core/ngx_crc32.c $NGX_ROOT/src/core/ngx_rbtree.c $VOD_ROOT/ngx_buffer_cache.c $VOD_ROOT/test/buffer_cache/main.c -o bctest -I $VOD_ROOT/test/buffer_cache -I $NGX_ROOT/src/core -I $NGX_ROOT/src/event -I $NGX_ROOT/src/event/modules -I $NGX_ROOT/src/os/unix -I $NGX_ROOT/objs -I $VOD_ROOT -g
 * ./bctest