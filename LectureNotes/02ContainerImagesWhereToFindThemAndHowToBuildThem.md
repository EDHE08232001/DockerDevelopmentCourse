# Containers Images

## In This Lecture
1. **All about images**
2. **What's in an image (And what isn't)**
3. **Using Docker Hub registry**
4. **Managing our local image cache**
5. **Building our own image**

## What is in an image? (And what isn't)
- **App binaries**
- **Metadata** about the image data and how to run the image
- **Official Definition**:  
  "An image is an ordered collection of root filesystem changes and the corresponding execution parameters for use within a container runtime."
- **Not a complete OS**:  
  No kernel (since the host provides it), no kernel modules (e.g., drivers)

