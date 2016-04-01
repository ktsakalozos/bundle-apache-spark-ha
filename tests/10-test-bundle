#!/usr/bin/python3

import amulet
import time
import yaml
import unittest
import os
import requests


class TestDeployment(unittest.TestCase):
    bundle_file = os.path.join(os.path.dirname(__file__), '..', 'bundle.yaml')

    def setUp(self):
        self.d = amulet.Deployment(series='trusty')
        with open(self.bundle_file) as f:
            bun = f.read()
        bundle = yaml.safe_load(bun)
        self.d.load(bundle)
        self.d.setup(timeout=1800)
        self.d.sentry.wait(timeout=1800)

    def test_master_selected(self):
        '''
        Wait for all three spark units to agree on a master leader.
        Remove the leader unit.
        Check that a new leader is elected.
        '''
        self.d.sentry.wait_for_messages({"spark": ["Ready (standalone HA)",
                                                   "Ready (standalone HA)",
                                                   "Ready (standalone HA)"]}, timeout=900)
        # Give some slack for the spark units to elect a master
        # time.sleep(60)
        master = ''
        masters_count = 0
        for unit in self.d.sentry['spark']:
            ip = unit.info['public-address']
            url = 'http://{}:8080'.format(ip)
            homepage = requests.get(url)
            if 'ALIVE' in homepage.text:
                masters_count += 1
                master = unit.info['unit_name']
            else:
                assert 'STANDBY' in homepage.text

        assert masters_count == 1

        print("Removing master: {} ".format(master))
        self.d.remove_unit(master)
        time.sleep(120)
 
        self.d.sentry.wait_for_messages({"spark": ["Ready (standalone HA)",
                                                   "Ready (standalone HA)"]}, timeout=900)

        masters_count = 0
        for unit in self.d.sentry['spark']:
            ip = unit.info['public-address']
            url = 'http://{}:8080'.format(ip)
            homepage = requests.get(url)
            if 'ALIVE' in homepage.text:
                print("New master is {}".format(unit.info['unit_name']))
                masters_count += 1
            else:
                assert 'STANDBY' in homepage.text

        assert masters_count == 1


if __name__ == '__main__':
    unittest.main()
