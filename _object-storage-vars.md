{%- set access_key = "253a3c7ca27f4731a9c757addfac29ca" -%}
{%- set secret_key = "be057f235abf45ee8e2ba14edc5fb253" -%}

{%- if "gov" in build_flags -%}
  {%- set file_suffix = "-gov" -%}
  {%- set identity_guide_url = "/nhncloud/ko/public-api/iaas-token-gov/" -%}
  {%- set base_region = "KR1" -%}
  {%- set identity_url = "https://api-identity-infrastructure.gov-nhncloudservice.com" -%}
  {%- set ec = false -%}
  {%- set encrypt = false -%}
  {%- set replication = true -%}
  {%- set ratelimit = true -%}
  {%- set terraform_support = true -%}
  {%- set terraform_guide_url = "/nhncloud/ko/terraform-guide-gov/" -%}
  {%- set regions = [
    {"code": "KR1", "name": "한국(판교) 리전", "endpoint": "https://kr1-api-object-storage.gov-nhncloudservice.com"},
    {"code": "KR2", "name": "한국(평촌) 리전", "endpoint": "https://kr2-api-object-storage.gov-nhncloudservice.com"},
  ] -%}

{%- elif "ncgn" in build_flags -%}
  {%- set file_suffix = "-ncgn" -%}
  {%- set identity_guide_url = "/nhncloud/ko/public-api/iaas-token-ncgn/" -%}
  {%- set base_region = "KR1" -%}
  {%- set identity_url = "https://api-identity-infrastructure.gncloud.go.kr" -%}
  {%- set ec = false -%}
  {%- set encrypt = false -%}
  {%- set replication = false -%}
  {%- set ratelimit = true -%}
  {%- set terraform_support = false -%}
  {%- set terraform_guide_url = "/Compute/Instance/ko/terraform-guide/" -%}
  {%- set regions = [
    {"code": "KR1", "name": "한국(판교) 리전", "endpoint": "https://api-object-storage.gncloud.go.kr"},
  ] -%}

{%- elif "ppp" in build_flags -%}
  {%- set base_region = "KR4" -%}
  {%- set ec = false -%}
  {%- set replication = false -%}
  {%- set ratelimit = false -%}
  {%- set terraform_support = false -%}
  {%- set terraform_guide_url = "/Compute/Instance/ko/terraform-guide/" -%}
  {%- if "ngcc" in build_flags -%}
    {%- set file_suffix = "-ngcc" -%}
    {%- set identity_guide_url = "/nhncloud/ko/public-api/iaas-token-ngcc/" -%}
    {%- set identity_url = "https://ngcc-kr4-iaas.kr.cloud.toastoven.net/identity" -%}
    {%- set encrypt = false -%}
    {%- set regions = [
      {"code": "KR4", "name": "한국(대구) 리전", "endpoint": "http://ngcc-kr4-swift.kr.cloud.toastoven.net"},
    ] -%}
  {%- elif "ninc" in build_flags -%}
    {%- set file_suffix = "-ninc" -%}
    {%- set identity_guide_url = "/nhncloud/ko/public-api/iaas-token-ninc/" -%}
    {%- set identity_url = "https://api-identity-infrastructure.ninc.go.kr" -%}
    {%- set encrypt = false -%}
    {%- set regions = [
      {"code": "KR4", "name": "한국(대구) 리전", "endpoint": "https://kr4-api-object-storage.ninc.go.kr"},
    ] -%}
  {%- elif "ngsc" in build_flags -%}
    {%- set file_suffix = "-ngsc" -%}
    {%- set identity_guide_url = "/nhncloud/ko/public-api/iaas-token-ngsc/" -%}
    {%- set identity_url = "https://api-identity-infrastructure.ngsc.go.kr" -%}
    {%- set encrypt = false -%}
    {%- set regions = [
      {"code": "KR4", "name": "한국(대구) 리전", "endpoint": "https://kr4-api-object-storage.ngsc.go.kr"},
    ] -%}
  {%- elif "ngoic" in build_flags -%}
    {%- set file_suffix = "-ngoic" -%}
    {%- set identity_guide_url = "/nhncloud/ko/public-api/iaas-token-ngoic/" -%}
    {%- set identity_url = "https://api-identity-infrastructure.ngoic.com" -%}
    {%- set encrypt = true -%}
    {%- set regions = [
      {"code": "KR4", "name": "한국(대구) 리전", "endpoint": "https://kr4-api-object-storage.ngoic.com"},
    ] -%}
  {%- elif "ngovc" in build_flags -%}
    {%- set file_suffix = "-ngovc" -%}
    {%- set identity_guide_url = "/nhncloud/ko/public-api/iaas-token-ngovc/" -%}
    {%- set identity_url = "https://api-identity-infrastructure.ngovc.com" -%}
    {%- set encrypt = true -%}
    {%- set regions = [
      {"code": "KR4", "name": "한국(대구) 리전", "endpoint": "https://kr4-api-object-storage.ngovc.com"},
    ] -%}
  {%- endif -%}

{%- else -%}
  {#- public (기본값) -#}
  {%- set file_suffix = "" -%}
  {%- set identity_guide_url = "/nhncloud/ko/public-api/iaas-token/" -%}
  {%- set base_region = "KR1" -%}
  {%- set identity_url = "https://api-identity-infrastructure.nhncloudservice.com" -%}
  {%- set ec = true -%}
  {%- set encrypt = true -%}
  {%- set replication = true -%}
  {%- set ratelimit = true -%}
  {%- set terraform_support = true -%}
  {%- set terraform_guide_url = "/nhncloud/ko/terraform-guide/" -%}
  {%- set regions = [
    {"code": "KR1", "name": "한국(판교) 리전", "endpoint": "https://kr1-api-object-storage.nhncloudservice.com"},
    {"code": "KR2", "name": "한국(평촌) 리전", "endpoint": "https://kr2-api-object-storage.nhncloudservice.com"},
    {"code": "KR3", "name": "한국(광주) 리전", "endpoint": "https://kr3-api-object-storage.nhncloudservice.com"},
    {"code": "JP1", "name": "일본(도쿄) 리전", "endpoint": "https://jp1-api-object-storage.nhncloudservice.com"},
  ] -%}
{%- endif -%}

{%- set object_storage_url = (regions | selectattr('code', 'eq', base_region) | first).endpoint -%}
