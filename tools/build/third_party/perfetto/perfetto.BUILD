# Copyright (C) 2019 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

load("@gapid//tools/build:rules.bzl", "cc_copts", "cc_stripped_binary", "copy_tree")
load("@gapid//tools/build/third_party/perfetto:protozero.bzl", "cc_protozero_library")
load("@gapid//tools/build/third_party/perfetto:ipc.bzl", "cc_ipc_library")
load("@io_bazel_rules_go//proto:def.bzl", "go_proto_library")
load("@io_bazel_rules_go//go:def.bzl", "go_library")

_COPTS_BASE = cc_copts() + [
    "-DPERFETTO_IMPLEMENTATION",
    #    # Always build in optimized mode.
    "-O2",
    "-DNDEBUG",
] + select({
    "@gapid//tools/build:windows": ["-D__STDC_FORMAT_MACROS"],
    "//conditions:default": [],
})

_COPTS = _COPTS_BASE + [
    "-DPERFETTO_BUILD_WITH_EMBEDDER",
]

cc_library(
    name = "base",
    srcs = [
        "src/base/event.cc",
        "src/base/string_splitter.cc",
        "src/base/string_view.cc",
        "src/base/string_utils.cc",
        "src/base/thread_checker.cc",
        "src/base/time.cc",
        "src/base/virtual_destructors.cc",
        "src/base/paged_memory.cc",
    ] + glob([
        "include/**/*.h",
    ]) + select({
        "@gapid//tools/build:windows": [],
        "@gapid//tools/build:darwin": [],
        "@gapid//tools/build:linux": [
            "src/base/file_utils.cc",
            "src/base/metatrace.cc",
            "src/base/pipe.cc",
            "src/base/temp_file.cc",
            "src/base/unix_socket.cc",
            "src/base/unix_task_runner.cc",
        ],
        # Android
        "//conditions:default": [
            "src/base/android_task_runner.cc",
            "src/base/file_utils.cc",
            "src/base/metatrace.cc",
            "src/base/pipe.cc",
            "src/base/temp_file.cc",
            "src/base/unix_socket.cc",
        ],
    }),
    hdrs = glob(["include/**/*.h"]),
    copts = _COPTS,
    strip_include_prefix = "include",
    visibility = ["//visibility:public"],
)

cc_library(
    name = "base_ipc",
    srcs = [
        "src/ipc/client_impl.h",
        "src/ipc/client_impl.cc",
        "src/ipc/virtual_destructors.cc",
        "src/ipc/service_proxy.cc",
        "src/ipc/deferred.cc",
        "src/ipc/buffered_frame_deserializer.h",
        "src/ipc/host_impl.cc",
        "src/ipc/buffered_frame_deserializer.cc",
        "src/ipc/host_impl.h",
    ] + glob([
        "include/**/*.h",
    ]),
    hdrs = glob(["include/**/*.h"]),
    copts = _COPTS,
    strip_include_prefix = "include",
    visibility = ["//visibility:public"],
    deps = [
        ":base",
        ":wire_protocol_cc_proto",
    ],
)

cc_library(
    name = "protozero",
    srcs = [
        "src/protozero/message.cc",
        "src/protozero/message_handle.cc",
        "src/protozero/proto_decoder.cc",
        "src/protozero/scattered_heap_buffer.cc",
        "src/protozero/scattered_stream_null_delegate.cc",
        "src/protozero/scattered_stream_writer.cc",
    ],
    hdrs = glob(["include/perfetto/protozero/*.h"]),
    copts = _COPTS,
    strip_include_prefix = "include",
    visibility = ["//visibility:public"],
    deps = [
        ":base",
    ],
)

cc_library(
    name = "trace_processor",
    srcs = [
        "src/trace_processor/android_logs_table.cc",
        "src/trace_processor/args_table.cc",
        "src/trace_processor/args_tracker.cc",
        "src/trace_processor/clock_tracker.cc",
        "src/trace_processor/counter_definitions_table.cc",
        "src/trace_processor/counter_values_table.cc",
        "src/trace_processor/event_tracker.cc",
        "src/trace_processor/filtered_row_index.cc",
        "src/trace_processor/ftrace_descriptors.cc",
        "src/trace_processor/ftrace_utils.cc",
        "src/trace_processor/fuchsia_provider_view.cc",
        "src/trace_processor/fuchsia_trace_parser.cc",
        "src/trace_processor/fuchsia_trace_tokenizer.cc",
        "src/trace_processor/fuchsia_trace_utils.cc",
        "src/trace_processor/heap_profile_allocation_table.cc",
        "src/trace_processor/heap_profile_callsite_table.cc",
        "src/trace_processor/heap_profile_frame_table.cc",
        "src/trace_processor/heap_profile_mapping_table.cc",
        "src/trace_processor/heap_profile_tracker.cc",
        "src/trace_processor/instants_table.cc",
        "src/trace_processor/metrics/metrics.cc",
        "src/trace_processor/process_table.cc",
        "src/trace_processor/process_tracker.cc",
        "src/trace_processor/proto_trace_parser.cc",
        "src/trace_processor/proto_trace_tokenizer.cc",
        "src/trace_processor/query_constraints.cc",
        "src/trace_processor/raw_table.cc",
        "src/trace_processor/row_iterators.cc",
        "src/trace_processor/sched_slice_table.cc",
        "src/trace_processor/slice_table.cc",
        "src/trace_processor/slice_tracker.cc",
        "src/trace_processor/span_join_operator_table.cc",
        "src/trace_processor/sql_stats_table.cc",
        "src/trace_processor/sqlite3_str_split.cc",
        "src/trace_processor/stats_table.cc",
        "src/trace_processor/storage_columns.cc",
        "src/trace_processor/storage_schema.cc",
        "src/trace_processor/storage_table.cc",
        "src/trace_processor/string_pool.cc",
        "src/trace_processor/string_table.cc",
        "src/trace_processor/syscall_tracker.cc",
        "src/trace_processor/table.cc",
        "src/trace_processor/thread_table.cc",
        "src/trace_processor/trace_processor.cc",
        "src/trace_processor/trace_processor_context.cc",
        "src/trace_processor/trace_processor_impl.cc",
        "src/trace_processor/trace_sorter.cc",
        "src/trace_processor/trace_storage.cc",
        "src/trace_processor/virtual_destructors.cc",
        "src/trace_processor/window_operator_table.cc",
    ] + glob([
        "include/**/*.h",
        "src/trace_processor/**/*.h",
    ]) + [
        ":sql_metrics_h",
    ],
    hdrs = glob(["include/**/*.h"]),
    copts = _COPTS + ["-Iexternal/perfetto/sqlite"],
    strip_include_prefix = "include",
    visibility = ["//visibility:public"],
    deps = [
        ":base",
        ":common_zero_proto",
        ":metrics_zero_proto",
        ":protozero",
        ":trace_zero_proto",
        "//sqlite",
        "@com_google_protobuf//:protobuf",
    ],
)

genrule(
    name = "sql_metrics_h",
    srcs = [
        "src/trace_processor/metrics/android/android_mem.sql",
        "src/trace_processor/metrics/android/android_mem_lmk.sql",
    ],
    outs = [
        "src/trace_processor/metrics/sql_metrics.h",
    ],
    cmd = "$(location :gen_merged_sql_metrics) --cpp_out=$@ $(SRCS)",
    tools = [
        ":gen_merged_sql_metrics",
    ],
)

py_binary(
    name = "gen_merged_sql_metrics",
    srcs = ["tools/gen_merged_sql_metrics.py"],
)

cc_binary(
    name = "protozero_plugin",
    srcs = [
        "src/protozero/protoc_plugin/protozero_generator.cc",
        "src/protozero/protoc_plugin/protozero_generator.h",
        "src/protozero/protoc_plugin/protozero_plugin.cc",
    ],
    copts = _COPTS,
    deps = [
        "@com_google_protobuf//:protoc_lib",
    ],
)

cc_binary(
    name = "ipc_plugin",
    srcs = [
        "src/ipc/protoc_plugin/ipc_generator.cc",
        "src/ipc/protoc_plugin/ipc_generator.h",
        "src/ipc/protoc_plugin/ipc_plugin.cc",
    ],
    copts = _COPTS,
    deps = [
        "@com_google_protobuf//:protoc_lib",
    ],
)

proto_library(
    name = "common_proto",
    srcs = [
        "perfetto/common/android_log_constants.proto",
        "perfetto/common/commit_data_request.proto",
        "perfetto/common/observable_events.proto",
        "perfetto/common/sys_stats_counters.proto",
        "perfetto/common/trace_stats.proto",
    ],
)

proto_library(
    name = "wire_protocol_proto",
    srcs = [
        "src/ipc/wire_protocol.proto",
    ],
)

cc_proto_library(
    name = "wire_protocol_cc_proto",
    deps = [":wire_protocol_proto"],
)

proto_library(
    name = "ipc_proto",
    srcs = [
        "perfetto/ipc/consumer_port.proto",
        "perfetto/ipc/producer_port.proto",
    ],
    deps = [
        ":common_proto",
        ":config_proto",
    ],
)

cc_proto_library(
    name = "ipc_cc_proto",
    visibility = ["//visibility:public"],
    deps = [":ipc_proto"],
)

cc_proto_library(
    name = "common_cc_proto",
    visibility = ["//visibility:public"],
    deps = [":common_proto"],
)

cc_library(
    name = "core",
    srcs = [
        "src/tracing/core/trace_buffer.cc",
        "src/tracing/core/shared_memory_abi.cc",
        "src/tracing/core/packet_stream_validator.h",
        "src/tracing/core/process_stats_config.cc",
        "src/tracing/core/test_config.cc",
        "src/tracing/core/data_source_descriptor.cc",
        "src/tracing/core/null_trace_writer.cc",
        "src/tracing/core/trace_writer_for_testing.h",
        "src/tracing/core/id_allocator.h",
        "src/tracing/core/trace_writer_impl.h",
        "src/tracing/core/packet_stream_validator.cc",
        "src/tracing/core/heapprofd_config.cc",
        "src/tracing/core/virtual_destructors.cc",
        "src/tracing/core/trace_config.cc",
        "src/tracing/core/commit_data_request.cc",
        "src/tracing/core/null_trace_writer.h",
        "src/tracing/core/sliced_protobuf_input_stream.cc",
        "src/tracing/core/data_source_config.cc",
        "src/tracing/core/sys_stats_config.cc",
        "src/tracing/core/ftrace_config.cc",
        "src/tracing/core/startup_trace_writer_registry.cc",
        "src/tracing/core/id_allocator.cc",
        "src/tracing/core/trace_buffer.h",
        "src/tracing/core/trace_stats.cc",
        "src/tracing/core/observable_events.cc",
        "src/tracing/core/android_power_config.cc",
        "src/tracing/core/startup_trace_writer.cc",
        "src/tracing/core/shared_memory_arbiter_impl.h",
        "src/tracing/core/android_log_config.cc",
        "src/tracing/core/shared_memory_arbiter_impl.cc",
        "src/tracing/core/sliced_protobuf_input_stream.h",
        "src/tracing/core/android_log_constants.cc",
        "src/tracing/core/chrome_config.cc",
        "src/tracing/core/tracing_service_impl.cc",
        "src/tracing/core/trace_writer_impl.cc",
        "src/tracing/core/patch_list.h",
        "src/tracing/core/sys_stats_counters.cc",
        "src/tracing/core/tracing_service_impl.h",
        "src/tracing/core/trace_packet.cc",
        "src/tracing/core/inode_file_config.cc",
    ] + glob([
        "include/**/*.h",
    ]),
    hdrs = glob(["include/**/*.h"]),
    copts = _COPTS,
    strip_include_prefix = "include",
    visibility = ["//visibility:public"],
    deps = [
        ":base",
        ":common_cc_proto",
        ":config_cc_proto",
        ":trace_cc_proto",
        ":trace_zero_proto",
        "@com_google_googletest//:gtest_main",
    ],
)

cc_library(
    name = "ipc",
    srcs = [
        "src/tracing/ipc/consumer/consumer_ipc_client_impl.cc",
        "src/tracing/ipc/consumer/consumer_ipc_client_impl.h",
        "src/tracing/ipc/default_socket.cc",
        "src/tracing/ipc/default_socket.h",
        "src/tracing/ipc/posix_shared_memory.cc",
        "src/tracing/ipc/posix_shared_memory.h",
        "src/tracing/ipc/producer/producer_ipc_client_impl.cc",
        "src/tracing/ipc/producer/producer_ipc_client_impl.h",
        "src/tracing/ipc/service/consumer_ipc_service.cc",
        "src/tracing/ipc/service/consumer_ipc_service.h",
        "src/tracing/ipc/service/producer_ipc_service.cc",
        "src/tracing/ipc/service/producer_ipc_service.h",
        "src/tracing/ipc/service/service_ipc_host_impl.cc",
        "src/tracing/ipc/service/service_ipc_host_impl.h",
    ] + glob([
        "include/**/*.h",
    ]),
    hdrs = glob(["include/**/*.h"]),
    copts = _COPTS,
    strip_include_prefix = "include",
    visibility = ["//visibility:public"],
    deps = [
        ":base",
        ":core",
        ":ipc_cc_ipc",
        ":ipc_cc_proto",
    ],
)

cc_protozero_library(
    name = "common_zero_proto",
    copts = _COPTS,
    deps = [":common_proto"],
)

cc_ipc_library(
    name = "ipc_cc_ipc",
    cdeps = [
        ":ipc_cc_proto",
        ":base_ipc",
    ],
    copts = _COPTS,
    deps = [":ipc_proto"],
)

proto_library(
    name = "config_proto",
    srcs = [
        "perfetto/config/android/android_log_config.proto",
        "perfetto/config/chrome/chrome_config.proto",
        "perfetto/config/data_source_config.proto",
        "perfetto/config/data_source_descriptor.proto",
        "perfetto/config/ftrace/ftrace_config.proto",
        "perfetto/config/inode_file/inode_file_config.proto",
        "perfetto/config/power/android_power_config.proto",
        "perfetto/config/process_stats/process_stats_config.proto",
        "perfetto/config/profiling/heapprofd_config.proto",
        "perfetto/config/sys_stats/sys_stats_config.proto",
        "perfetto/config/test_config.proto",
        "perfetto/config/trace_config.proto",
    ],
    visibility = ["//visibility:public"],
    deps = [
        ":common_proto",
    ],
)

proto_library(
    name = "config_combined_proto",
    srcs = ["perfetto/config/perfetto_config.proto"],
    visibility = ["//visibility:public"],
)

go_proto_library(
    name = "config_go_proto",
    importpath = "perfetto/config",
    proto = ":config_combined_proto",
    visibility = ["//visibility:public"],
)

java_proto_library(
    name = "config_java_proto",
    visibility = ["//visibility:public"],
    deps = [":config_combined_proto"],
)

proto_library(
    name = "metrics_proto",
    srcs = [
        "perfetto/metrics/android/mem_metric.proto",
        "perfetto/metrics/metrics.proto",
    ],
)

cc_protozero_library(
    name = "metrics_zero_proto",
    copts = _COPTS,
    deps = [":metrics_proto"],
)

proto_library(
    name = "trace_proto",
    srcs = [
        "perfetto/trace/android/android_log.proto",
        "perfetto/trace/android/packages_list.proto",
        "perfetto/trace/chrome/chrome_trace_event.proto",
        "perfetto/trace/clock_snapshot.proto",
        "perfetto/trace/filesystem/inode_file_map.proto",
        "perfetto/trace/ftrace/binder.proto",
        "perfetto/trace/ftrace/block.proto",
        "perfetto/trace/ftrace/cgroup.proto",
        "perfetto/trace/ftrace/clk.proto",
        "perfetto/trace/ftrace/compaction.proto",
        "perfetto/trace/ftrace/ext4.proto",
        "perfetto/trace/ftrace/f2fs.proto",
        "perfetto/trace/ftrace/fence.proto",
        "perfetto/trace/ftrace/filemap.proto",
        "perfetto/trace/ftrace/ftrace.proto",
        "perfetto/trace/ftrace/ftrace_event.proto",
        "perfetto/trace/ftrace/ftrace_event_bundle.proto",
        "perfetto/trace/ftrace/ftrace_stats.proto",
        "perfetto/trace/ftrace/generic.proto",
        "perfetto/trace/ftrace/i2c.proto",
        "perfetto/trace/ftrace/ipi.proto",
        "perfetto/trace/ftrace/irq.proto",
        "perfetto/trace/ftrace/kmem.proto",
        "perfetto/trace/ftrace/lowmemorykiller.proto",
        "perfetto/trace/ftrace/mdss.proto",
        "perfetto/trace/ftrace/mm_event.proto",
        "perfetto/trace/ftrace/oom.proto",
        "perfetto/trace/ftrace/power.proto",
        "perfetto/trace/ftrace/raw_syscalls.proto",
        "perfetto/trace/ftrace/regulator.proto",
        "perfetto/trace/ftrace/sched.proto",
        "perfetto/trace/ftrace/signal.proto",
        "perfetto/trace/ftrace/sync.proto",
        "perfetto/trace/ftrace/task.proto",
        "perfetto/trace/ftrace/test_bundle_wrapper.proto",
        "perfetto/trace/ftrace/vmscan.proto",
        "perfetto/trace/ftrace/workqueue.proto",
        "perfetto/trace/interned_data/interned_data.proto",
        "perfetto/trace/power/battery_counters.proto",
        "perfetto/trace/power/power_rails.proto",
        "perfetto/trace/profiling/profile_packet.proto",
        "perfetto/trace/ps/process_stats.proto",
        "perfetto/trace/ps/process_tree.proto",
        "perfetto/trace/sys_stats/sys_stats.proto",
        "perfetto/trace/system_info.proto",
        "perfetto/trace/test_event.proto",
        "perfetto/trace/trace.proto",
        "perfetto/trace/trace_packet.proto",
        "perfetto/trace/track_event/debug_annotation.proto",
        "perfetto/trace/track_event/process_descriptor.proto",
        "perfetto/trace/track_event/task_execution.proto",
        "perfetto/trace/track_event/thread_descriptor.proto",
        "perfetto/trace/track_event/track_event.proto",
        "perfetto/trace/trigger.proto",
        "perfetto/trace/trusted_packet.proto",
    ],
    deps = [
        ":common_proto",
        ":config_proto",
    ],
)

cc_protozero_library(
    name = "trace_zero_proto",
    copts = _COPTS,
    deps = [":trace_proto"],
)

proto_library(
    name = "perfetto_cmd_proto",
    srcs = [
        "src/perfetto_cmd/descriptor.proto",
        "src/perfetto_cmd/perfetto_cmd_state.proto",
    ],
)

cc_proto_library(
    name = "perfetto_cmd_cc_proto",
    visibility = ["//visibility:public"],
    deps = [":perfetto_cmd_proto"],
)

cc_proto_library(
    name = "config_cc_proto",
    visibility = ["//visibility:public"],
    deps = [":config_proto"],
)

cc_proto_library(
    name = "trace_cc_proto",
    deps = [":trace_proto"],
)

cc_stripped_binary(
    name = "perfetto_cmd",
    srcs = [
        "src/perfetto_cmd/config.cc",
        "src/perfetto_cmd/config.h",
        "src/perfetto_cmd/main.cc",
        "src/perfetto_cmd/pbtxt_to_pb.cc",
        "src/perfetto_cmd/pbtxt_to_pb.h",
        "src/perfetto_cmd/perfetto_cmd.cc",
        "src/perfetto_cmd/perfetto_cmd.h",
        "src/perfetto_cmd/perfetto_config.descriptor.h",
        "src/perfetto_cmd/rate_limiter.cc",
        "src/perfetto_cmd/rate_limiter.h",
        "src/perfetto_cmd/trigger_perfetto.cc",
        "src/perfetto_cmd/trigger_producer.cc",
        "src/perfetto_cmd/trigger_producer.h",
    ],
    copts = _COPTS,
    visibility = ["//visibility:public"],
    deps = [
        ":base",
        ":config_cc_proto",
        ":ipc",
        ":perfetto_cmd_cc_proto",
        ":protozero",
        ":trace_cc_proto",
        "@com_google_protobuf//:protobuf",
    ],
)

cc_stripped_binary(
    name = "traced",
    srcs = [
        "src/base/watchdog_posix.cc",
        "src/traced/service/lazy_producer.cc",
        "src/traced/service/lazy_producer.h",
        "src/traced/service/main.cc",
        "src/traced/service/service.cc",
    ],
    copts = _COPTS_BASE,
    linkopts = select({
        "@gapid//tools/build:linux": ["-lrt"],
        "@gapid//tools/build:darwin": [],
        "@gapid//tools/build:windows": [],
    }),  # keep
    visibility = ["//visibility:public"],
    deps = [
        ":base",
        ":ipc",
        ":perfetto_cmd_cc_proto",
        ":protozero",
        ":trace_cc_proto",
        "@com_google_protobuf//:protobuf",
    ],
)

cc_stripped_binary(
    name = "traced_probes",
    srcs = [
        "src/traced/probes/main.cc",
        "src/traced/probes/probes_data_source.cc",
        "src/traced/probes/probes_data_source.h",
        "src/traced/probes/probes_producer.cc",
        "src/traced/probes/probes_producer.h",
        "src/traced/probes/probes.cc",
        "src/traced/probes/ftrace/atrace_wrapper.h",
        "src/traced/probes/ftrace/event_info.cc",
        "src/traced/probes/ftrace/proto_translation_table.h",
        "src/traced/probes/ftrace/page_pool.cc",
        "src/traced/probes/ftrace/ftrace_procfs.h",
        "src/traced/probes/ftrace/atrace_wrapper.cc",
        "src/traced/probes/ftrace/page_pool.h",
        "src/traced/probes/ftrace/ftrace_stats.h",
        "src/traced/probes/ftrace/ftrace_metadata.cc",
        "src/traced/probes/ftrace/event_info.h",
        "src/traced/probes/ftrace/ftrace_config_muxer.cc",
        "src/traced/probes/ftrace/ftrace_config.h",
        "src/traced/probes/ftrace/format_parser.cc",
        "src/traced/probes/ftrace/ftrace_thread_sync.h",
        "src/traced/probes/ftrace/ftrace_stats.cc",
        "src/traced/probes/ftrace/ftrace_config.cc",
        "src/traced/probes/ftrace/cpu_stats_parser.h",
        "src/traced/probes/ftrace/atrace_hal_wrapper.h",
        "src/traced/probes/ftrace/page_pool_unittest.cc",
        "src/traced/probes/ftrace/event_info_constants.h",
        "src/traced/probes/ftrace/proto_translation_table.cc",
        "src/traced/probes/ftrace/ftrace_data_source.cc",
        "src/traced/probes/ftrace/cpu_reader.h",
        "src/traced/probes/ftrace/atrace_hal_wrapper.cc",
        "src/traced/probes/ftrace/ftrace_controller.cc",
        "src/traced/probes/ftrace/ftrace_procfs.cc",
        "src/traced/probes/ftrace/ftrace_config_muxer.h",
        "src/traced/probes/ftrace/cpu_reader.cc",
        "src/traced/probes/ftrace/ftrace_data_source.h",
        "src/traced/probes/ftrace/event_info_constants.cc",
        "src/traced/probes/ftrace/ftrace_controller.h",
        "src/traced/probes/ftrace/ftrace_metadata.h",
        "src/traced/probes/ftrace/format_parser.h",
        "src/traced/probes/ftrace/cpu_stats_parser.cc",
        "src/traced/probes/filesystem/inode_file_data_source.cc",
        "src/traced/probes/filesystem/fs_mount.cc",
        "src/traced/probes/filesystem/prefix_finder.h",
        "src/traced/probes/filesystem/lru_inode_cache.h",
        "src/traced/probes/filesystem/file_scanner.h",
        "src/traced/probes/filesystem/lru_inode_cache.cc",
        "src/traced/probes/filesystem/range_tree.cc",
        "src/traced/probes/filesystem/prefix_finder.cc",
        "src/traced/probes/filesystem/inode_file_data_source.h",
        "src/traced/probes/filesystem/file_scanner.cc",
        "src/traced/probes/filesystem/fs_mount.h",
        "src/traced/probes/filesystem/range_tree.h",
        "src/android_internal/atrace_hal.h",
        "src/android_internal/health_hal.h",
        "src/android_internal/power_stats_hal.h",
        "src/traced/probes/android_log/android_log_data_source.cc",
        "src/traced/probes/android_log/android_log_data_source.h",
        "src/traced/probes/packages_list/packages_list_data_source.cc",
        "src/traced/probes/packages_list/packages_list_data_source.h",
        "src/traced/probes/power/android_power_data_source.cc",
        "src/traced/probes/power/android_power_data_source.h",
        "src/traced/probes/ps/process_stats_data_source.cc",
        "src/traced/probes/ps/process_stats_data_source.h",
        "src/traced/probes/sys_stats/sys_stats_data_source.cc",
        "src/traced/probes/sys_stats/sys_stats_data_source.h",
        "src/base/watchdog_posix.cc",
    ] + select({
        "@gapid//tools/build:linux": [],
        "@gapid//tools/build:windows": [],
        "@gapid//tools/build:darwin": [],
        # Android
        "//conditions:default": [
            "src/android_internal/atrace_hal.cc",
            "src/android_internal/power_stats_hal.cc",
            "src/android_internal/health_hal.cc",
        ],
    }),
    copts = _COPTS_BASE,
    linkopts = select({
        "@gapid//tools/build:linux": [
            "-ldl",
            "-lrt",
        ],
        "@gapid//tools/build:darwin": [],
        "@gapid//tools/build:windows": [],
    }),  # keep
    visibility = ["//visibility:public"],
    deps = [
        ":base",
        ":common_cc_proto",
        ":common_zero_proto",
        ":ipc",
        ":perfetto_cmd_cc_proto",
        ":protozero",
        ":trace_zero_proto",
        "@com_google_googletest//:gtest_main",
        "@com_google_protobuf//:protobuf",
    ],
)