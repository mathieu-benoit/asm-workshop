# asm-workshop

Put this https://alwaysupalwayson.com/asm-security as a workshop.

1. [ ] Create a GKE cluster
1. [ ] Install ASM
1. [ ] Ingress Gateway
1. [ ] Egress Gateway
1. [ ] Install OnlineBoutique
1. [ ] mTLS
1. [ ] Sidecar
1. [ ] AuthorizationPolicies
1. [ ] NetworkPolicies
1. [ ] Policy Controller
1. [ ] Monitoring: Topology, SLOs, Traces, etc.
1. [ ] Misc: any Istio's features about traffic management, etc.

Further considerations:
- Do the same with BankOfAnthos?
- Multi-cluster?
- MCP (control/data plane)?
- Integrate CRfA in there? Or do another similar crfa-workshop?


## Build and run this static web site locally

```
git clone --recurse-submodules https://github.com/mathieu-benoit/asm-workshop
cd asm-workshop
docker build -t asm-workshop .
docker run -d -p 8080:8080 asm-workshop
```

## Configure GitHub action

FIXME