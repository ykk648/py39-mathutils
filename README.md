### Notes

Update [mathutils](https://pypi.org/project/mathutils/) to support python3.9.

Already apply py39 patch from [official link](https://gitlab.com/ideasman42/blender-mathutils/uploads/aebb580ffe731f0d0b56d9dc7463b5c6/Py39-PyNoArgsFucntion-20210809.patch)

```sh
git clone https://github.com/ykk648/py39-mathutils.git
cd py39-mathutils
python setup.py build
sudo python setup.py install
```

### Old Readme

Ref [old_readme](./old_readme.rst)
