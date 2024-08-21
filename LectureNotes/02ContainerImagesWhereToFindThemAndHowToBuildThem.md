# Containers Images

## What is in an image? (And what isn't)
- **App binaries**
- **Metadata** about the image data and how to run the image
- **Official Definition**:  
  "An image is an ordered collection of root filesystem changes and the corresponding execution parameters for use within a container runtime."
- **Not a complete OS**:  
  No kernel (since the host provides it), no kernel modules (e.g., drivers)
- **Small as one file (your app binary)** like golang static binary
- **Big** as a Ubuntu distro with apt, and apache, PHP, and more installed