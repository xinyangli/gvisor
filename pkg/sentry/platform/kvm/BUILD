load("//tools:defs.bzl", "arch_genrule", "go_library", "go_test", "select_arch")
load("//tools/go_generics:defs.bzl", "go_template_instance")
load("//tools/nogo:defs.bzl", "nogo_facts")

package(licenses = ["notice"])

go_template_instance(
    name = "atomicptr_machine",
    out = "atomicptr_machine_unsafe.go",
    package = "kvm",
    prefix = "machine",
    template = "//pkg/sync/atomicptr:generic_atomicptr",
    types = {
        "Value": "machine",
    },
)

nogo_facts(
    name = "bluepill_impl",
    srcs = [
        "//pkg/ring0:aarch64.go",
        "//pkg/ring0:defs.go",
        "//pkg/ring0:defs_amd64.go",
        "//pkg/ring0:defs_arm64.go",
        "//pkg/ring0:entry_template.go",
        "//pkg/ring0:x86.go",
    ],
    output = "bluepill_impl.s",
    template = select_arch(
        amd64 = "bluepill_amd64.s",
        arm64 = "bluepill_arm64.s",
    ),
    deps = [
        "//pkg/abi/linux",
        "//pkg/cpuid",
        "//pkg/hostarch",
        "//pkg/ring0/pagetables",
        "//pkg/sentry/arch",
        "//pkg/sentry/arch/fpu",
    ],
)

arch_genrule(
    name = "bluepill_impl_arch",
    src = ":bluepill_impl",
    template = "bluepill_impl_%s.s",
)

go_library(
    name = "kvm",
    srcs = [
        "address_space.go",
        "address_space_amd64.go",
        "address_space_arm64.go",
        "atomicptr_machine_unsafe.go",
        "bluepill.go",
        "bluepill_allocator.go",
        "bluepill_amd64.go",
        "bluepill_amd64_unsafe.go",
        "bluepill_arm64.go",
        "bluepill_arm64_unsafe.go",
        "bluepill_fault.go",
        "bluepill_unsafe.go",
        "context.go",
        "filters.go",
        "filters_amd64.go",
        "filters_arm64.go",
        "kvm.go",
        "kvm_amd64.go",
        "kvm_amd64_unsafe.go",
        "kvm_arm64.go",
        "kvm_arm64_unsafe.go",
        "kvm_const.go",
        "kvm_const_arm64.go",
        "machine.go",
        "machine_amd64.go",
        "machine_amd64_unsafe.go",
        "machine_arm64.go",
        "machine_arm64_unsafe.go",
        "machine_unsafe.go",
        "physical_map.go",
        "physical_map_amd64.go",
        "physical_map_arm64.go",
        "virtual_map.go",
        ":bluepill_impl_arch",
    ],
    visibility = ["//pkg/sentry:internal"],
    deps = [
        "//pkg/abi/linux",
        "//pkg/atomicbitops",
        "//pkg/context",
        "//pkg/cpuid",
        "//pkg/hostarch",
        "//pkg/log",
        "//pkg/metric",
        "//pkg/procid",
        "//pkg/ring0",
        "//pkg/ring0/pagetables",
        "//pkg/seccomp",
        "//pkg/sentry/arch",
        "//pkg/sentry/arch/fpu",
        "//pkg/sentry/memmap",
        "//pkg/sentry/platform",
        "//pkg/sentry/platform/interrupt",
        "//pkg/sentry/time",
        "//pkg/sighandling",
        "//pkg/sync",
        "@org_golang_x_sys//unix:go_default_library",
    ],
)

go_test(
    name = "kvm_test",
    srcs = [
        "kvm_amd64_test.go",
        "kvm_amd64_test.s",
        "kvm_arm64_test.go",
        "kvm_safecopy_test.go",
        "kvm_test.go",
        "virtual_map_test.go",
    ],
    library = ":kvm",
    # FIXME(gvisor.dev/issue/3374): Not working with all build systems.
    nogo = False,
    # cgo has to be disabled. We have seen libc that blocks all signals and
    # calls mmap from pthread_create, but we use SIGSYS to trap mmap system
    # calls.
    pure = True,
    tags = [
        "manual",
        "nogotsan",
        "requires-kvm",
    ],
    deps = [
        "//pkg/abi/linux",
        "//pkg/cpuid",
        "//pkg/hostarch",
        "//pkg/memutil",
        "//pkg/ring0",
        "//pkg/ring0/pagetables",
        "//pkg/safecopy",
        "//pkg/sentry/arch",
        "//pkg/sentry/arch/fpu",
        "//pkg/sentry/platform",
        "//pkg/sentry/platform/kvm/testutil",
        "//pkg/sentry/time",
        "@org_golang_x_sys//unix:go_default_library",
    ],
)
