# ![NGINX logo](https://raw.github.com/g17/registry-ldap-auth/master/images/NginxLogo.gif)

## Introduction

This image provides an LDAP authentication proxy for a [Docker registry](https://github.com/docker/docker-registry). Therefore it uses an [NGINX web server](https://github.com/nginx/nginx) with builtin [LDAP support](https://github.com/kvspb/nginx-auth-ldap) and SSL. It is based on [h3nrik/nginx-ldap](https://registry.hub.docker.com/u/h3nrik/nginx-ldap/).

## Prerequisites

The authentication proxy works with different LDAP servers like ApacheDS or OpenLDAP. It also works with Active Directory.

If you need information about creating a container with a test LDAP server please refer to [h3nrik/nginx-ldap](https://registry.hub.docker.com/u/h3nrik/nginx-ldap/).

## Installation

Assuming you have running containers for the Docker registry named *registry* and an LDAP server named *ldap*. The following steps will add LDAP authentication to your registry.

1. A valid SSL certificate must be copied into a local folder (e.g. /ssl/cert/path) to be mounted as a volume into the proxy server later. It must be a valid one known by a trusted CA! The certificate file must be named *docker-registry.crt* and the private key file *docker-registry.key*.

2. Create an LDAP configuration file named *ldap.conf*. A [sample-ldap.conf](/sample-ldap.conf) file is provided with the image sources. It could look like:

		url ldap://ldap/dc=example,dc=com?samaccountname?sub?(objectClass=user);
		binddn ldap@example.com;
		binddn_passwd secretPassword;
		group_attribute uniquemember;
		group_attribute_is_dn on;
		require group 'cn=docker,ou=groups,dc=example,dc=com';
		require valid_user;
		satisfy all;	

3. Create a Docker container for the authentication proxy. The proxy container expects the registry host name to be named *docker-registry*. The used NGINX web server configuration can be found [in the config folder](/config/).

		docker run --name registry-ldap-auth --link ldap:ldap --link registry:docker-registry -v /ssl/cert/path:/etc/ssl/docker:ro -v `pwd`/ldap.conf:/etc/nginx/ldap.conf:ro -p 80:80 -p 443:443 -p 5000:5000 -d h3nrik/registry-ldap-auth

Theoretically you could also use self-signed certificates. Therefore the Docker daemon need to be started with the *--insecure-registry* command line parameter. But this is not recommended.

## Licenses

This docker image contains compiled binaries for:

1. The NGINX web server. Its license can be found on the [NGINX website](http://nginx.org/LICENSE).
2. The nginx-auth-ldap module. Its license can be found on the [nginx-auth-ldap module project site](https://github.com/kvspb/nginx-auth-ldap/blob/master/LICENSE).