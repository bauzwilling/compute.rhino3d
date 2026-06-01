# compute.rhino3d - Public Fork

This is a public fork of the [McNeel compute.rhino3d](https://github.com/mcneel/compute.rhino3d) project featuring enhancements related to request decompression and geometry endpoint improvements.

## Key Commits Included

This fork includes the following commits:

### 1. Implement Request Decompression (7ecfab5)
**Date:** April 22, 2026  
**Author:** Mostafa Nouh

**Description:** Implements request decompression functionality to handle compressed HTTP requests in the compute services.

**Files Modified:**
- `src/compute.geometry/Startup.cs` - Added decompression middleware configuration
- `src/rhino.compute/ReverseProxy.cs` - Enhanced reverse proxy to support decompression
- `src/rhino.compute/Startup.cs` - Integrated decompression startup logic

**Impact:** Enables the compute services to efficiently handle compressed requests, reducing bandwidth usage and improving performance for clients sending large payloads.

### 2. Implement Request Compression (1621e4b)
**Date:** April 22, 2026  
**Author:** Mostafa Nouh

**Description:** Implements gzip request body compression in the compute client to complement server-side decompression, achieving up to 5x performance improvement for large payloads.

**Files Modified:**
- `src/compute.geometry/RhinoCompute.cs` - Added gzip request compression and response decompression support

**Impact:** Reduces bandwidth usage significantly (up to 5x faster transfer times) and improves overall request/response performance, especially for large geometry payloads.

### 3. Fix Issue of the Unroller on Rhino Compute (12c0792)
**Date:** April 22, 2026  
**Author:** Mostafa Nouh

**Description:** Fixes critical `AccessViolationException` issues in the geometry unroller endpoint on Rhino Compute. The unroller component in custom geometry endpoints was throwing protected memory access violations because it lacked a proper headless RhinoDoc context.

**Root Cause:** The `GeometryEndPoint.Post()` method was not respecting the `Config.CreateHeadlessDoc` configuration setting, unlike the Grasshopper solver helper module. This caused unroller operations to fail when attempting to access protected Rhino memory without a proper document context.

**The Fix:** 
```csharp
Rhino.RhinoDoc createdDoc = null;

try
{
    if (Config.CreateHeadlessDoc)
    {
        createdDoc = Rhino.RhinoDoc.CreateHeadless(null);
        Rhino.RhinoDoc.ActiveDoc = createdDoc;
    }
    // ... Process unroller and other geometry operations ...
}
finally
{
    if (createdDoc != null)
    {
        createdDoc.Dispose();
    }
}
```

**Files Modified:**
- `src/compute.geometry/GeometryEndPoint.cs` - Added headless document creation logic (65 insertions, 46 deletions)

**Impact:** 
- Resolves `System.AccessViolationException: Attempted to read or write protected memory` errors
- Enables reliable unroller operations on custom geometry endpoints
- Ensures proper memory cleanup with document disposal
- Maintains consistency with Grasshopper solver module behavior
- Fixes critical issue reported in [McNeel Forum](https://discourse.mcneel.com/t/unroller-accessviolationexception-in-rhino-compute/213235)

### 4. Fork Documentation (c26f311)
**Date:** June 1, 2026  
**Author:** Mostafa Nouh

**Description:** Comprehensive documentation of the fork including technical details on compression implementation, IIS configuration guidance, and detailed explanation of the unroller fix.

**Files Modified:**
- `FORK_DOCUMENTATION.md` - Complete fork documentation with setup and configuration guides

**Impact:** Provides developers with clear guidance on using the fork, configuring IIS for compression, and understanding the improvements included.

## About This Project

Rhino Compute is a service that exposes Rhino's computational geometry as REST APIs. It allows developers to leverage Rhino's powerful geometry engine through web services.

### Project Structure

- **compute.geometry/** - Core geometry computation engine
- **rhino.compute/** - REST API server and reverse proxy
- **hops/** - Grasshopper Hops plugin integration
- **ghhops-server-py/** - Python server implementation for Grasshopper Hops

### Features

- REST APIs for 3D geometry operations
- Request decompression support for efficient data transfer
- Grasshopper Hops integration for visual programming
- Python server support
- Both synchronous and asynchronous operations

## Getting Started

### Prerequisites

- .NET Core SDK
- Rhino (for full functionality)
- Git

### Building

```bash
# Clone this fork
git clone https://github.com/yourusername/compute.rhino3d.git
cd compute.rhino3d/src

# Build the solution
dotnet build

# For Release build
dotnet build -c Release
```

### Running

```bash
# Run compute.geometry service
cd compute.geometry
dotnet run

# Or run rhino.compute proxy
cd ../rhino.compute
dotnet run
```

## Technical Details

### Request Compression
The implementation adds support for gzip-compressed request bodies and automatic gzip/deflate response decompression in the compute client. This can significantly improve request/response performance, often yielding up to 5x faster transfer times for large payloads.

This change is implemented in `src/compute.geometry/RhinoCompute.cs` by:
- setting `HttpWebRequest.AutomaticDecompression` for GZip and Deflate
- adding `Content-Encoding: gzip` to outgoing requests
- compressing the JSON payload using `GZipStream`

### IIS Compression Configuration
To enable response compression in IIS:
1. Open **IIS Manager**.
2. Select the server node or the target website.
3. Open the **Compression** feature.
4. Enable **Static content compression** and **Dynamic content compression**.
5. Restart IIS or recycle the application pool.

You can also configure IIS in `web.config`:

```xml
<system.webServer>
  <urlCompression doStaticCompression="true" doDynamicCompression="true" />
  <httpCompression>
    <dynamicTypes>
      <add mimeType="application/json" enabled="true" />
    </dynamicTypes>
  </httpCompression>
</system.webServer>
```

For request compression, IIS does not automatically decompress incoming gzip request bodies by default, so the application must handle them. In this fork, the compute client sends gzip-compressed JSON requests and the server must accept `Content-Encoding: gzip` headers. When using IIS as a reverse proxy to Kestrel, ensure IIS forwards request bodies unchanged and does not reject the `Content-Encoding` header.

### Geometry Unroller Fixes
The critical fix addresses the `AccessViolationException` that occurred when calling unroller operations on custom geometry endpoints. The solution ensures that:
- A proper headless RhinoDoc context is created when `Config.CreateHeadlessDoc` is enabled
- The `RhinoDoc.ActiveDoc` is properly set for unroller operations
- Protected Rhino memory is safely accessed through the document context
- Resources are properly cleaned up with document disposal in the finally block

This aligns the GeometryEndPoint behavior with the Grasshopper solver module, which already implemented this pattern.

## License

This fork retains the original license from the McNeel compute.rhino3d project. See the LICENSE file in the compute.geometry directory for details.

## Contributing

For changes, please:
1. Create a feature branch
2. Make your changes with clear commit messages
3. Test thoroughly
4. Submit a pull request

## References

- Original Repository: https://github.com/mcneel/compute.rhino3d
- Grasshopper Hops: https://www.grasshopper3d.com/
- Rhino Developer: https://developer.rhino3d.com/

## Author

**Mostafa Nouh**  
Based on the McNeel Rhino Compute project

---

*Last updated: June 1, 2026*
