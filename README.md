# HOU Share
The purpose of this repository is to share and save some test and examples from
Sidefx Houdini. There is no predefined order to it, just project file and small description.
All files are in Houdini Indie Edition so extension is .hiplc  not .hip
if VEX code will be essential, code will be include in README.md file.

### Traffic v01
Its a basic Traffic loop on path. Its done in sop solver using VEX in Wrangle.
Cars randomly spread along path drive in one direction, slow and stop in front of obstacles and other cars.
Detection is quite primitive

![Traffic_v01_cam1](https://github.com/szawel/HOU_Share/blob/master/gif/Traffic_v01_cam1.gif)
![Traffic_v01_cam2](https://github.com/szawel/HOU_Share/blob/master/gif/Traffic_v01_cam2.gif)


### Wrangle
Wrangle_v01.hiplc

#### Point Connection
point connection using Wrangle Sop

```C++
// point connection
int pr = addprim( geoself(), "polyline" );

addvertex( geoself(), pr, 0 );

for ( int i = 0; i < @numpt; i++ ){
    addvertex( geoself(), pr, i );
};
```
#### Path Extention
Extend path on both ends, by adding one points for each end.
![Traffic_v01_cam2](https://github.com/szawel/HOU_Share/blob/master/gif/wrangle_path_extension.gif)
copy and paste to VEXpression window in wrangle Sop.
```C++
// path | line | curve exten tool
float ex_len = ch( "ex_len" );                      // length control
vector pos[];                                       // empty array

// calculate extendet end poins
vector near_dir = normalize( point(0,"P",0) - point(0,"P",1) );
vector near_end_P = point(0,"P",0) + ( near_dir * ex_len );
vector far_dir = normalize( point(0,"P",@numpt-1) - point(0,"P",@numpt-2) );
vector far_end_P = point(0,"P",@numpt-1) + ( far_dir * ex_len );

// fill array
insert(pos,0,far_end_P);
for( int i = 0; i<@numpt; i++){
    vector ptP = point(0,"P",i);
    insert(pos,i,ptP);
}
insert(pos,0,near_end_P);

// remove all prim and point
removeprim(geoself(),0,0);
removepoint(geoself(),@ptnum);

// add nev prim
int pr = addprim(geoself(), "polyline");
// Start the curve with our point
addvertex(geoself(), pr, @ptnum);
if(@ptnum==0){
    for (int i = 0; i < len(pos); i++)
    {
        int pt = addpoint(geoself(), i);
        // write a new position
        setpointattrib(geoself(), "P", pt, pos[i]);
        // Connect the new point to our curve.
        addvertex(geoself(), pr, pt);
    }
}
```

### Camera Focus Distance
Calculate the distance to object from camera "focus_target"
```C++
vlength(vtorigin(".","../focus_target"))
```

### Redshift Instances from file

```C++
int nr = fit01(rand(@ptnum),0,24);
string filebase = chs("rs_without_fnum", 0);
string filerest = sprintf(".%02d.rs", nr);
s@instancefile = concat(filebase, filerest);
``` 

### Redshift Instances from file v2 

```C++
f@pscale = 0.001;

int rnum = ceil(rand(@id) * 17);

s@instance = "/obj/Shell" + itoa(rnum);
```

### Random Variant Attribute

```C++
int nr = chi("nr");
float seed = chf("seed");

i@variant = rint(fit01(rand(@ptnum+seed),0,nr));
```

### FBX Sequence

In top shelf right click on an empty area `New Tool` select `Script` tab
and copy code below
```
node = hou.node("/path_to_fbx_export_operator")
```
```C++
import hou

node = hou.node("/obj/geo1/rop_fbx2")

for i in range(1,13):
    hou.setFrame(i)
    node.render() 
```


### Select polygon by verex count
```
primvertexcount(0, @primnum) <= 3
primvertexcount(0, @primnum) == 4
primvertexcount(0, @primnum) > 4
```

### Scale rig individual bones by it local origin
set-up `axis` to `1,1,1`

```
float amount = chf("amount");
amount = $PI*amount;
vector axis = chv('axis');

@scale = quaternion(amount, axis);
```


### Houdini and ffmpeg

```
PDG_IMAGEMAGICK = "your path here"
PDG_FFMPEG = "your path here"
```


### Extra
ffmpeg gif from sequence
```Bash
ffmpeg -f image2 -i in_%005d.png out.gif
```


Markdown | Less | Pretty
--- | --- | ---
`%` | modulus | remainder on integer division. `5 % 2` results in `14 % 2` results in 0
`&&` | AND | logical operator `(x > 1) && (x < 10)` when both are true, it is true must have a complete condition on each side
` Pipe * 2` | OR | logical operator `(x > 1) PipePipe (x < 10)` when both are false, it is false, otherwise true(on the keyboard it is located above the enter key)
`>=``<=` | - | greater than or equal(Do not reverse the order –won’t work)less than or equal
`<``>` | - | less than greater than
`==` | equal | is something equal –this is because = is for assignment== is a comparison this is by far the hardest thing to get used to for new users
`!=not` | equal | - |
` `` `| backticks | - | to convert an expression into a string ie. `ch(“../someparameter”)` in the font node would print that verbatim, but putting this i backticks would give me the value
`if` | if | syntax is odd in Houdini if ( condition, then part, else part )for example in a switch statement you might have if ($F > 0, 1, 0)note the thepart or the else part could be another if statementie. if ( $F > 0, if ( $F > 2, 1, 0 ), 0 )
`ifs` | - | as above but returns a string,for example `ifs($F>1,”Hi”,”There”)`


### Python script

``` Python
import hou

def create_nested_nodes():
    # Check if a node is selected
    if len(hou.selectedNodes()) != 1:
        print("Please select a single node.")
        return

    # Get the selected node
    selected_node = hou.selectedNodes()[0]

    # Ensure the selected node is in the geometry context
    if selected_node.type().category().name() != "Sop":
        print("Selected node is not in the geometry context.")
        return

    # Evaluate geometry to extract points
    geo = selected_node.geometry()

    # Get the parent (object level) node
    obj_node = selected_node.parent()

    # Create a subnet (or use existing) for the nested nodes
    subnet_name = "nested_nodes"
    if subnet_name in [child.name() for child in obj_node.children()]:
        subnet = obj_node.node(subnet_name)
    else:
        subnet = obj_node.createNode("subnet", subnet_name)

    # Create a node for each point
    for point in geo.points():
        point_id = point.number()
        # Name the node based on point id
        node_name = "point_{}".format(point_id)
        new_node = subnet.createNode("null", node_name)  # Change "geo" to "null"
        new_node.moveToGoodPosition()

        # Here you can adjust the position, attributes, etc. of each node based on the point data

    # Layout the nodes nicely
    subnet.layoutChildren()

create_nested_nodes()

```