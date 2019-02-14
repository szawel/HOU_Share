# HOU Share
The purpose of this repository is to share and save some test and examples from
sidefx Houdini. There is no predefined order to it, just project file and small description.
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
extend path
![Traffic_v01_cam2](https://github.com/szawel/HOU_Share/blob/master/gif/wrangle_path_extension.gif)
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


### Extra
ffmpeg gif from sequence
```Bash
ffmpeg -f image2 -i .\in_%005d.png out.gif
```
