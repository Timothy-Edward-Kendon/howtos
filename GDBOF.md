# Macros available in GDB OpenFoam

| name                  | info                                                                                                                                |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| pPatchList (pPl)      | Prints a list with the patch name and its ID                                                                                        |
| pInternalValues (pIv) | Prints the field values of an internal mesh on Geometric Field T                                                                    |
| pPatchValues (pPv)    | Prints the field values on the selected patch                                                                                       |
| pfvMatrixFull         | Print in a file the full fvMatrix to load in octave/matlab                                                                          |
| pfvMatrixSparse       | Print in a file the full fvMatrix to load in octave/matlab in sparse format                                                         |
| pInternalValuesLimits | Prints the values of the internal mesh in the specified range of cells                                                              |
| pPatchValuesLimits    | Prints the values of the patch in the specified cells                                                                               |
| pFindCell             | Prints the nearest cell centroid index and the cell centroid index that contains the point specified                                |
| pFindFace             | Prints the nearest patchID from the point, and the nearest faceID in this patch                                                     |
| pSurfaceValues        | Prints the values of the faces in the cell                                                                                          |
| pExportFoamFormat     | vol\*Field in Foam format. It allows converting it to VTK and opening with Paraview                                                 |
| pFieldValues          | Prints the field values without associated mesh (Every Field is considered like scalarField)                                        |
| pFieldValuesLimits    | Prints the values of the field without associated mesh in the specified range of cells (Every Field is considered like scalarField) |
| pName                 | Prints the name of the object                                                                                                       |
