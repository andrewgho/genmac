#!/usr/bin/env ruby
# genmac - generate valid looking random MAC address
# Andrew Ho (andrew@zeuscat.com)
#
# For more details, see the README.md that ships with this program, from:
# https://github.com/andrewgho/genmac
#
# Copyright (c) 2014, Andrew Ho
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
# 1. Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
#
# 3. Neither the name of Andrew Ho nor the names of its contributors may
#    be used to endorse or promote products derived from this software
#    without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
# IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED
# TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A
# PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED
# TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
# PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
# LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
# NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

require 'optparse'

ME = File.basename($0)
USAGE =
  "usage: #{ME} [-h] [-v] [-l] [-u] [-n num] [-d delim] [-f oui_file] [org]\n"
FULL_USAGE = USAGE + <<'end'
  -h        display this help text and exit
  -v        verbose mode, print debug messages and matching organizations
  -l        set locally administered bit (default universally administered)
  -u        output uppercase hex digits (default lowercase)
  -n num    generate this many MAC addresses (default 1)
  -d delim  set delimiter to separate hex octets (default colon)
  -f file   IEEE organizationally unique identifiers file (default oui.txt)
  org       generate MAC address from this organization (default random octets)
end

DEFAULT_OUI_FILE = File.join(File.dirname(__FILE__), 'oui.txt')
HEX_OCTET = %r{[0-9A-F][0-9A-F]}
OUI_OCTETS = %r{#{HEX_OCTET}-#{HEX_OCTET}-#{HEX_OCTET}}

$verbose = false

def main(argv = ARGV)
  locally_administered = false
  uppercase = false
  num_addresses = 1
  delimiter = ':'
  oui_file = DEFAULT_OUI_FILE
  OptionParser.new do |opts|
    opts.on('-h', '--help')           { puts FULL_USAGE; return 0 }
    opts.on('-v', '--verbose')        { $verbose = true }
    opts.on('-l', '--local')          { locally_administered = true }
    opts.on('-u', '--uppercase')      { uppercase = true }
    opts.on('-n NUM',  '--number')    { |arg| num_addresses = arg.to_i }
    opts.on('-d DELIM','--delimiter') { |arg| delimiter = arg }
    opts.on('-f FILE', '--file')      { |arg| oui_file = arg }
    begin
      opts.parse!(argv)
    rescue OptionParser::InvalidOption => e
      abort "#{ME}: #{e}"
    end
  end
  org = argv.empty? ? nil : argv.shift
  if org
    puts "Searching for organization #{org} in file #{oui_file}..." if $verbose
    oui_octets_list = find_matching_oui_octets(oui_file, org)
    if oui_octets_list.empty?
      abort "#{ME}: could not find organization matching: #{org}"
    end
  end
  format = uppercase ? '%02X' : '%02x'
  1.upto(num_addresses) do
    if org
      mac_octets = oui_octets_list[rand(oui_octets_list.size)].dup
      if locally_administered
        mac_octets[0] |= 0x02
      else
        mac_octets[0] &= ~0x02
      end
    else
      mac_octets = (1..3).collect { rand(256) }
      mac_octets[0] |= 0x02 if locally_administered
    end
    (1..3).each { mac_octets << rand(256) }
    mac = mac_octets.collect { |i| format % [i] }.join(delimiter)
    puts mac
  end
  0
end

def find_matching_oui_octets(filename, criteria = nil)
  selector =
    criteria.is_a?(String) ? lambda { |s| s.match(%r{^#{criteria}\b}i) } :
    criteria.is_a?(Regexp) ? lambda { |s| s.match(criteria) } :
    criteria.is_a?(Proc)   ? lambda { |s| criteria.call(s) } :
    block_given?           ? lambda { |s| yield(s) } :
                             lambda { |s| true }
  octets_list = []
  begin
    File.open(filename, 'r').each do |line|
      if (matchdata = line.match(/^  (#{OUI_OCTETS})\s*\(hex\)\s*(.+)$/))
        octets, organization = matchdata[1,2]
        if selector.call(organization)
          if $verbose
            if octets_list.empty?
              puts
              puts 'Octets   | Organization'
              puts '---------+-------------'
            end
            puts "#{octets} | #{organization}"
          end
          octets_list << octets.split(/-/).collect { |s| s.hex }
        end
      end
    end
    puts if $verbose && !octets_list.empty?
  rescue Errno::ENOENT => e
    abort "#{ME}: could not open organizationally unique identifiers file: #{e}"
  end
  octets_list
end

exit(main(ARGV))
