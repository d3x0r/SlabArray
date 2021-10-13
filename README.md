

# Slab Array (JS Package)

JS implementation of a fragmented array.



This is a key-value relation track; it's more like `Map()` than a hash; it relies on external users to generate their own values to be hashed.

This is best when fed entries that have a lot of change in the first bytes of the string to store; sequential inserts cause the most work.

See Also https://github.com/d3x0r/BloomNHash for C version and other tests.

This has hooks to support [object storage](https://github.com/d3x0r/sack.vfs/blob/master/README_ObjectStorage.md); 
includes hold off timer to flush writes; allowing rapid updates to perform quickly, while writing blocks of changes at once; 
reduces re-writing the same hash blocks that have similar changes quickly.

This implementation is async(promise based).

|method| args | return | description |
|---|----|----|----|
|(constructor) | (<optional storage>) | hash | An optional object storage instance can be passed to the constructor. |
|push| (string) | object | Returns a promise which resolves with the object stored in the hash at the specified string. |
|get| (index) | object | returns item stored in the array at specified index |
|length | <getter> | number | returns the number of items in the array |
| forEach | (cb)  |    | async foreach which awaits each callback.  |


## In memory example.

This code fragment demonstrates the above method descriptions.

``` js
import {SlabArray} from "slab-array"
{
	const array = new SlabArray();
        array.push( "asdf" );
        array.get( 0 ).then( (object)=>{
		console.log( "This probably got deleted before actually getting done, maybe?" );
	} );
}

```

## Example of usage with object storage.


This demonstrates hooking into storage; the fragments of the hash will automatically be tracked as stored objects.
`get` and `set` methods load externally stored entries sparsely.

A more complete example would demonstrate storing the object that is set in the hash map.  Objects that have
themselves been saved (or loaded) in object storage will have remote references stored in the hash blockks, and 
will refer to the external object instead.

``` js
import {ObjectStorage} from "ObjectStorage"
const storage = ObjectStorage( "hostAddress" ); // remote address providing storage
import {SlabArray} from "./slabarray.mjs"

```

``` js
import {ObjectStorage} from "@d3x0r/object-storage"
````


``` js
import {sack} from "sack.vfs"
const storage = sack.ObjectStorage( "Filename" );
import {SlabArray} from "./slabarray.mjs"

async function init() 
{
	SlabArray.hook( storage );

	let array = null;

	let root = await storage.getRoot();
	if( !root.find( "config" ) ){
		array = new SlabArray();
		let root   = await storage.getRoot();
		const file = await root.create( "config" );
		const id   = await storage.put( array ); // storing the hash 
		file.write( id );
	}else {
		const file = await root.open("config");
		array       = await storage.get( {id: file.read() } );
	}

	const already = await array.get( 0 );
	if( !already )
	        array.push( 1234 );
	else
		console.log( "Value is already:", already );

}

```


## Changelog

- 1.0.1
   - Change which object storage is used
- 1.0.0 - Initial Release (WIP)