# Copyright 2019 Nokia
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM alpine:3.9
MAINTAINER Balazs Szekeres <balazs.szekeres@nokia.com>

ARG KUBEDNS_VERSION
ENV GOPATH /build
ARG go_install_dir="/usr/local/go"
ARG go_version
ENV PATH="$go_install_dir/bin:$PATH"

RUN apk update 
RUN  apk upgrade 
RUN  apk add sudo 
RUN  apk add --update libcap 
RUN  adduser -u 145 -D -H -s /sbin/nologin kubedns 
RUN  mkdir -p /home/kubedns 
RUN  chown -R kubedns:kubedns /home/kubedns 
RUN  chown -R kubedns:kubedns /etc 
RUN  apk add --no-cache --virtual .build-deps build-base git mercurial go glide bash tar curl 
RUN  apk add --no-cache dnsmasq 
RUN  mkdir -p $go_install_dir 
RUN  curl -fsSL -k https://dl.google.com/go/go${go_version}.src.tar.gz | tar zx --strip-components=1 -C ${go_install_dir} 
RUN  cd ${go_install_dir}/src/ \
&&  ./make.bash 
RUN  mkdir -p $GOPATH/src/k8s.io/dns 
RUN  curl -fsSL -k https://github.com/kubernetes/dns/archive/${KUBEDNS_VERSION}.tar.gz | tar zx --strip-components=1 -C $GOPATH/src/k8s.io/dns 
RUN  cd $GOPATH/src/k8s.io/dns \
&&  PKG=k8s.io/dns ARCH=amd64 VERSION=${KUBEDNS_VERSION} ./build/build.sh 
RUN  mv $GOPATH/bin/linux_amd64/kube-dns /usr/bin/kube-dns 
RUN  mv $GOPATH/bin/linux_amd64/dnsmasq-nanny /usr/bin/dnsmasq-nanny 
RUN  setcap 'cap_net_bind_service=+ep' /usr/sbin/dnsmasq \
&&  apk del .build-deps \
&&  rm -rf $GOPATH \
&7  rm -rf $go_install_dir \
&&  rm -rf /build \
&&  rm -rf /root/.glide

USER kubedns
ENV HOME /home/kubedns
WORKDIR $HOME


